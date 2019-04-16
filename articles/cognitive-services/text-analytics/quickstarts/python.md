---
title: 'Quickstart: Using Python to call the Text Analytics API'
titleSuffix: Azure Cognitive Services
description: Get information and code samples to help you quickly get started using the Text Analytics API in Microsoft Cognitive Services on Azure.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.component: text-analytics
ms.topic: quickstart
ms.date: 04/09/2019
ms.author: aahi
---

# Quickstart for Text Analytics API with Python 

This walkthrough shows you how to analyze four different aspects of text documents using the Text Analytics SDK for Python.

* [Detect languages](#detect_language)
* [Analyze sentiment](#SentimentAnalysis)
* [Extract key phrases](#KeyPhraseExtraction)
* [Named entity recognition](#named_entity_recognition)

You can run this example as a Jupyter notebook on [MyBinder](https://mybinder.org) by clicking on the **Launch Binder** badge below.

[![Binder](https://mybinder.org/badge.svg)](https://mybinder.org/v2/gh/Microsoft/cognitive-services-notebooks/master?filepath=TextAnalytics.ipynb)

Refer to the Text Analytics service's [REST API documentation](https://go.microsoft.com/fwlink/?LinkID=759346) for a reference to the main features of the Text Analytics service.

## Prerequisites

This Quickstart requires Python 3.0 or later and the Text Analytics SDK module for Python. You can install the Text Analytics module with the following shell command.

```bash
python -m pip install azure-cognitiveservices-language-textanalytics
```

This also installs other modules that are required by the Text Analytics SDK, if you don't already have them.

If you are using your own Jupyter installation to run the code in a notebook, make sure the IPython kernel is up-to-date.

```bash
python -m pip install --upgrade IPython
```

> [!TIP]
>  While you could call the [HTTP endpoints](https://westus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v2-1/operations/56f30ceeeda5650db055a3c7) directly from Python, the SDK makes it much easier to use the service without having to worry about HTTP requests or JSON.
>
> A couple of useful links:
> - [SDK PyPi page](https://pypi.org/project/azure-cognitiveservices-language-textanalytics/)
> - [SDK code](https://github.com/Azure/azure-sdk-for-python)

[!INCLUDE cognitive-services-text-analytics-signup-requirements]

Make a note of the [endpoint and subscription key](../How-tos/text-analytics-how-to-access-key.md) associated with your subscription.

The code in this Quickstart is presented in short snippets. You can run it on Binder (or your own Jupyter notebook) by placing the cursor into a code block and pressing Control-Enter. You can also run the code by pasting each snippet at the Python command line.

Run the following code before running the snippets in other sections. Replace `subscription_key` below with a valid subscription key (in quote marks) and verify that the region in the `endpoint` URL corresponds to the one you used when setting up the service. (If you are using a free trial key, it's in the `westcentralus` region, so you don't need to change the URL.)

```python
from azure.cognitiveservices.language.textanalytics import TextAnalyticsClient, models
from msrest.authentication import CognitiveServicesCredentials

try:
    from IPython.display import HTML
    assert get_ipython().__class__.__name__ == "ZMQInteractiveShell"
except Exception:
    HTML = print    # simply print HTML if we're not in a Jupyter notebook

subscription_key = None
assert subscription_key

endpoint = "https://westcentralus.api.cognitive.microsoft.com"

client = TextAnalyticsClient(endpoint, CognitiveServicesCredentials(subscription_key))
```

This code:

* Detects whether it is being run in a Jupyter notebook. If not, the `HTML` class (which is used to insert HTML result into a Jupyter notebook) is set to a reference to the `print` command.
* Sets the subscription key and endpoint, then initializes a Text Analytics client using those variables.

## Detect language

The Text Analytics client's [`detect_language` method](https://docs.microsoft.com/en-us/python/api/azure-cognitiveservices-language-textanalytics/azure.cognitiveservices.language.textanalytics.text_analytics_client.textanalyticsclient?view=azure-python#detect-language-show-stats-none--documents-none--custom-headers-none--raw-false----operation-config-) detects the language of submitted text documents. A document is plain text in a supported language; it need not be in a file.

To reduce the number of calls involved in processing large numbers of documents, multiple documents may be submitted in a single `detect_language` call. The input to the method is a list of individual documents, each of which is represented by a `LanguageInput` instance.

As a `LanguageInput` object, each document has `id` and `text` attributes. The `text` attribute stores the text to be analyzed. The `id` attribute is a string that associates each result with its original document, and must be unique within the document set for each `detect_language` call. A sample list with three documents is defined below.

```python
language_docs = [ models.LanguageInput(id="1", text="This is a document written in English."),
                  models.LanguageInput(id="2", text="Este es un document escrito en Español."),
                  models.LanguageInput(id="3", text='这是一个用中文写的文件')
                ]

language_results = client.detect_language(documents=language_docs)
```

After the `detect_language()` call returns, `language_results.documents` is a list in the same order as `language_docs`. For each original document, a `LanguageBatchResultItem` is provided, containing information about the language or languages detected in the document. 

Each result item contains a `detected_languages` attribute that holds a list of `DetectedLanguage` objects, each corresponding to a language found in the document. The `name` attribute of this object contains the human-readable name of the language, such as `English`, and the `score` attribute contains how certain the Text Analytics service is of the result on a scale from 0.0 to 1.0.

The following Python code generates an HTML table showing the original text and the detected language or languages, along with each language's score.

```python
table = []
header = "<tr><th>{}</th><th>{}</th><th>{}</th></tr>".format("ID", "Text", "Languages (scores)")

for doc, res in zip(language_docs, language_results.documents):
    langs = ", ".join("{} ({})".format(lang.name.replace("_", " "), lang.score) for lang in res.detected_languages)
    row = "<tr><td>{doc.id}</td><td>{doc.text}</td><td>{langs}</td></tr>".format(doc=doc, langs=langs)
    table.append(row)

HTML("<table>{0}{1}</table>".format(header, "\n".join(table)))
```

## Analyze sentiment

The [`sentiment` method](https://docs.microsoft.com/en-us/python/api/azure-cognitiveservices-language-textanalytics/azure.cognitiveservices.language.textanalytics.text_analytics_client.textanalyticsclient?view=azure-python#sentiment-show-stats-none--documents-none--custom-headers-none--raw-false----operation-config-) detects the sentiment of text documents, on a scale of 0.0 (unfavorable) to 1.0 (favorable). Values around 0.5 represent neutral sentiment.

In practice, a sentiment analysis call works much like a language detection call. Multiple pieces of text ("documents") can be submitted in a single call, and each document must have a unique ID within the set of documents in a given `sentiment` call. 

You must also specify the language of each document using ISO 639 standard language codes, such as `en` for English. The `MultiLanguageInput` class holds the required information about each document. 

The following example scores four documents, two in English and *dos* in Spanish.

```python
sentiment_docs = [ 
    models.MultiLanguageInput(id="1", language="en", 
        text="I had a wonderful experience! The rooms were wonderful and the staff was helpful."),
    models.MultiLanguageInput(id="2", language="en", 
        text="I had a terrible time at the hotel. The staff was rude and the food was awful."),
    models.MultiLanguageInput(id="3", language="es", 
        text="Los caminos que llevan hasta Monte Rainier son espectaculares y hermosos."),
    models.MultiLanguageInput(id="4", language="es", 
        text="La carretera estaba atascada. Había mucho tráfico el día de ayer."),
]

sentiment_results = client.sentiment(documents=sentiment_docs)
```

After the `sentiment` call, `sentiment_results.documents` is a list of `SentimentBatchResultItem` instances, each corresponding to a submitted document. The `SentimentBatchResultItem` includes a `score` attribute, which is the detected sentiment value. The Python code below displays the sentiment results as an HTML table.

```python
table = []
header = "<tr><th>{}</th><th>{}</th><th>{}</th></tr>".format("ID", "Text", "Score")

for doc, res in zip(sentiment_docs, sentiment_results.documents):
    row = "<tr><td>{doc.id}</td><td>{doc.text}</td><td>{score:.3}</td></tr>".format(doc=doc, score=res.score)
    table.append(row)

HTML("<table>{0}{1}</table>".format(header, "\n".join(table)))
```


## Extract key phrases

The [`key_phrases` method](https://docs.microsoft.com/en-us/python/api/azure-cognitiveservices-language-textanalytics/azure.cognitiveservices.language.textanalytics.text_analytics_client.textanalyticsclient?view=azure-python#key-phrases-show-stats-none--documents-none--custom-headers-none--raw-false----operation-config-) extracts key phrases from a text document.

A key phrase extraction call works much like a sentiment analysis call. Multiple "document can be submitted in a single call, and each document must have a unique ID within the set of documents in a given `key_phrases` call. 

You must specify the language of each document using ISO 639 standard language codes, such as `en` for English. The `MultiLanguageInput` class holds the required information about each document. 

We'll use the same documents we used for sentiment analysis in this example: four documents, half in English and half in Spanish. Here's the code to pass them to the `key_phrases` method.

```python
key_phrases_docs = [ 
    models.MultiLanguageInput(id="1", language="en", 
        text="I had a wonderful experience! The rooms were wonderful and the staff was helpful."),
    models.MultiLanguageInput(id="2", language="en", 
        text="I had a terrible time at the hotel. The staff was rude and the food was awful."),
    models.MultiLanguageInput(id="3", language="es", 
        text="Los caminos que llevan hasta Monte Rainier son espectaculares y hermosos."),
    models.MultiLanguageInput(id="4", language="es", 
        text="La carretera estaba atascada. Había mucho tráfico el día de ayer."),
]

key_phrases_results = client.key_phrases(documents=key_phrases_docs)
```

Much as we've seen with other methods, after the `key_phrases` call, `key_phrases_results.documents` is a list of `KeyPhraseBatchResultItem` instances, each corresponding to a submitted document. The `KeyPhraseBatchResultItem` has a `key_phrases` attribute, which is the detected sentiment value. The Python code below displays the results as an HTML table.

```python
table = []
header = "<tr><th>{}</th><th>{}</th><th>{}</th></tr>".format("ID", "Text", "Key phrases")

for doc, res in zip(key_phrases_docs, key_phrases_results.documents):
    phrases = ",".join(res.key_phrases)
    row = "<tr><td>{doc.id}</td><td>{doc.text}</td><td>{phrases}</td></tr>".format(doc=doc, phrases=phrases)
    table.append(row)

HTML("<table>{0}{1}</table>".format(header, "\n".join(table)))
```

## Named entity recognition

Finally, the [`entities` method](https://docs.microsoft.com/en-us/python/api/azure-cognitiveservices-language-textanalytics/azure.cognitiveservices.language.textanalytics.text_analytics_client.textanalyticsclient?view=azure-python#entities-show-stats-none--documents-none--custom-headers-none--raw-false----operation-config-) identifies entities (businesses, people, places, and other proper nouns) in a text document.

The overall process is familiar. Multiple documents can be submitted in a single call, and each document must have a unique ID within the set of documents in a given `entities` call. 

You must specify the language of each document using ISO 639 standard language codes, such as `en` for English. The `MultiLanguageInput` class stores the required information about each document. 

As before, here's our document set (this time just in English) and our Python method call.

```python
entity_docs = [ 
    models.MultiLanguageInput(id="1", language="en", text="I really enjoy the new XBox One S. "
        "It has a clean look, it has 4K/HDR resolution and it is affordable."),
    models.MultiLanguageInput(id="2", language="en", 
        text="The Seattle Seahawks won the Super Bowl in 2014.")
]

entity_results = client.entities(documents=entity_docs)
```

Once more, `entity_results.documents` is a list of `EntitiesBatchResultItem` instances corresponding to the submitted documents. The `entities` attribute of each object is a list of `EntityRecord` objects, each describing an entity recognized in the original document. 

There are several attributes of interest on an `EntityRecord` object, including its Bing ID, which can be used to retrieve more information about the entity. In this example, we'll use `name` (the entity's formal name),  `type` (its type), and `matches` (information about the parts of the document that were matched as each entity).

The following Python code produces an HTML table containing each recognized entity's formal name, its type, and its matches' text and location within the original document.

```python
table = []
header = "<tr><th>{}</th><th>{}</th><th>{}</th></tr>".format("ID", "Text", "Entities found")

for doc, res in zip(entity_docs, entity_results.documents):
    entities = "<p>".join("{} ({}): {}".format(e.name, e.type, 
        ", ".join("'{}' in chars {} thru {}".format(m.text, m.offset, m.offset + m.length - 1)
        for m in e.matches)) for e in res.entities)
    row = "<tr><td>{doc.id}</td><td>{doc.text}</td><td>{entities}</td></tr>".format(doc=doc, entities=entities)
    table.append(row)

HTML("<table>{0}{1}</table>".format(header, "\n".join(table)))
```


## Next steps

> [!div class="nextstepaction"]
> [Text Analytics With Power BI](../tutorials/tutorial-power-bi-key-phrases.md)

## See also 

 [Text Analytics overview](../overview.md)  
 [Frequently asked questions (FAQ)](../text-analytics-resource-faq.md)
