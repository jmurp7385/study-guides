# Machine Learning

## Rekognition

- find **object, people, text, and scenes** in **images and videos** using ML
- **Facial Analysis and facial search** to do user verification, people counting
- create a database of "familiar faces" or compare against celebrities
- Use Cases
  - labeling
  - content moderation
  - text detection
  - Face Detection and Analysis (gender, age range, emotions, ...)
  - Face Search and Verification
  - celebrity recognition
  - pathing (ex:m for sports game analysis)
- Content Moderation
  - detect content that is inappropriate, unwanted, or offensive (image and videos)
  - used in socuial media, broadcast media, advertising, and e-commerce situations to create a safer user experience
  - **set a minimum confidence threshold for items that will be flagged**
  - flag sensitive content for manual review in Amazon Augmented AI (A2I)
  - help comply with regulations

## Transcribe

- automatically **convert speech to text**
- uses a deep learning process called **automatic speech recognition (ASR)** to convert speech to text quickly and accurately
- **automatically remove Personally Identifiable Information PII using Redaction**
- **supports automatic language identification for multi-lingual audio**
- use cases
  - transcribe customer service calls
  - automate closed captioning and subtitles
  - generate metadata for media assets to create a fully searchable archive

## Polly

- turn text into lifelike speech using deep learning
- allowing you create applications that talk
- Lexicon & SSML
  - customize the pronunciation of words with Pronunciation Lexicons
    - stylized words: St3ph4ne -> Stephane
    - acronyms: AWS -> Amazon Web Services
  - upload lexicons andn use them in the SynthesizeSpeech Operation
  - generate speech from plain text or from documents marked up with **Speech Synthesis Markup Language (SSML)** - enables more customization
    - emphasizing specific words or phrases
    - using phonetic pronunciation
    - including breathing sounds, whispering
    - using the Newscaster speaking style

## Translate

- natural and accurate **language translation**
- allows you to **localize content** - such as websites and applications - for **international users** and to easily translate large volumes of text efficiently

## Lex + Connect

- Amazon Lex (same technology that powers Alexa)
  - **Automatic Speech Recognition (ASR)** to convert speech to text
  - Natural Language Understanding to recognize the intent of text, callers
  - helps build chatbots, call center bots
- Amazon Connect
  - recieve calls, create contact flows, cloud-based **virtual contact center**
  - can integratee with other CRM systems or AWS
  - no upfront payments, 80% cheaper than traditional contact center solutions

## Comprehend

- **Natural Language Processing**
- fully managed and serverless service
- uses machine learning to find insights anmd relationships in text
  - language of the text
  - extracts key phrases, places, people, brands, or events
  - understands how positive or negative the text is
  - analyzes using tokenization and parts of speech
  - automatically organizes a collection of text files by topic
- Use Cases:
  - analyze customer interactions (emails) to find what leasds to a positive or negative experience
  - create and group articles by topics tha Comprehend will uncover

## Comprehend Medical

- Amazon Comprehend Medical detects and returns useful information in unstructured clinical text
  - physician's notes
  - discharge summaries
  - test results
  - case notes
- **uses NLP to detect Protected Health Information (PHI)** - DetectPHI API
- store documents in Amazon S3, analyze real-time data with Kinesis Data Firehose, or use Amazon Transcrive to transcrive patient narratives into text that can be analyzed by Amazon Comprehend Medical

## SageMaker

- fully managed service for developers/data scientists to build ML models
- typically difficulty to do all the processes in one place + provision servers
- Machine learning process (simplified): predicting your exam score
  - historical data
    - \# years IT experience
    - \# years experience with AWS
    - time spent on course
  - Label data (with actual scores)
  - Build model
    - results in ML Model
  - train & tune model
  - Apply model on new Data

## Forecast

- fully managed service that uses ML to deliver highly accurate forecasts
- example: predict future sales of a raincoat
- 50% more accurate thatn looking at the data itself
- reduce forcasting time from months to hours
- use cases
  - product demand planning
  - financial planning
  - resource planning

## Kendra

- fully managed **document search service** powered by ML
- extract answers from within a document (text, pdf, HTML, PowerPoint, MS Word, FAQs,...)
- natural language search capabilities
- learn from user interactions/feedback to promote preferred results (**Incremental Learning**)
- ability to manually fine-tune search results (importance of data, freshness, custom)

## Personalize

- Fully managed ML-service to build apps with a real-time personalized recommendations
- example: personalized product recommendations/re-ranking, customized direct marketing
  - example: user bought gardening tools, provide recommendations on the next on to buy
- same technology used by <amazon.com>
- integrates into existing websites, applications, SMS, email marketing systems
- implement in days, not months (you dont need to build, train, and deploy ML solutions)
- use case: retail stores, media, and entertainment

## Textract

- automaticaly extracts text, handwriting, and data from any scanned documents using AI and ML
- extract data from forms and tables
- read and process any type of document (PDFs, Images)
- use cases:
  - financial services - invoices, financial reports
  - healthcare - medical records, insurance claims
  - public sector - tax forms, ID documents, passports

## Summary

- **Rekognition**: face detection, labeling, celebrity recognition
- **Transcribe**: audio to text (ex: subtitles)
- **Polly**: text to audio
- **Translate**: translations
- **Lex**: build converstational bots - chatbots
- **Connect**: cloud contact center
- **Comprehend**: natural language processing
- **SageMaker**: machine learning for every devloper/data scientist
- **Forecast**: build highly accurate forecasts
- **Kendra**: ML-powered document search engine
- **Personalize**: real-time personalized recommendations
- **Textract**: detect text and data in documents
