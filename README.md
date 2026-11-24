# When some peace never becomes (ack)knowledged: Towards the coloniality of non-knowledge in peace and conflict studies.

This library presentes a detailed methodology and empirical results of the paper [**When some peace never becomes (ack)knowledged: Towards the coloniality of non-knowledge in peace and conflict studies**](link), published in 31,506.


## 👥 Who is this for?
- **Researchers from the social sciences** that would like to perform bibliometrics or text analyses.
- **Social computer sciences** that would like to replicate this study in other domains than peace and conflict.
- **Policymakers and analysts** who would like to get involved in research policies to increase the participation of non-meanstream perspectives in the studies of peace and conflict.

## ✨ About

Through an analysis of 31,506 peace and conflict studies articles (1958-2024), this paper develops a novel framework for examining the epistemic conditions that prevent certain understandings of peace from becoming objects of knowledge. We argue that this stems not only from racialized, gendered, and Western-centric hierarchies, but from a coloniality of non-knowledge that selectively and structurally ignores particular actors, contexts, and histories. This shapes who is recognized as a legitimate knowledge producer and whose visions of peace matter. By foregrounding non-knowledge, we reveal how epistemic exclusions can constrain opportunities for peace for all.

## 🚀 Highlights
- 📈 Open Alex datasets about multilingual research on peace & conflict available for reproducibility 
- ⚖️ Full code to reproduce BERT based text labelling on concepts of peace and conflict.

## Table of Contents

1. [Methodology overview](#overview)
2. [Data labeling](#labeling)
3. [Human verification](#humans)
4. [Citation analyses](#citations)

## Methodology overview <a name="overview">

Flow chart with the steps to download the most worldwide diverse literature on Open Alex by different languages of publication and regions of the world for peace & conflict literature.

<p align="left">
  <img src="Methodology_flow_diagram.png" alt=“MethodsFlowChart" width=“500" style="vertical-align: middle; margin-right: 10px;"/>
</p>

An exhaustive list of the files from which each step is taking place is as follows: 1) translated terms peace & conflict is in this [table](Translated_query.csv) for 86+ lenagues; 2) files derived from the manual search of the translated terms in Open Alex after manual verification of the top results and topic displayed by the platform is downloaded and stored in this [folder](Multi-lingual-data); 3) Filtering those papers with an abstract and detection of language of each abstract and translation in this [jupter notebook](Multilingual_abstracts_translation.ipynb) and the corresponding translation files are in this [folder](List_translated_abstracts) and in this [file](Multilingual_journal_information.ipynb) is the process to download information of the journals of the papers. In this file is the [jupyter notebook](Multilingual_text_analyses.ipynb) merge of translated abstracts and this [file](https://www.dropbox.com/scl/fi/m2kdsy3atbiri96cwhp8x/Filtered_DataFrame_out_filt_new.pkl?rlkey=neemmgxqlrltxaayxjx61t3ct&st=07500ze1&dl=0) contains the entire final dataset for continuing the analyses.

## Data labeling <a name="labeling">
The entire labelling process can be found in this [jupyter notebook](Multilingual_text_analyses.ipynb) but desagreggated we:
1. We searched for the explicit appearance of each of the 30 dictionary terms in each ```['english_abstract', 'primary_topic.display_name’]``` previously defined by by Open Alex.
2. We split each translated abstract in sentences with ```from nltk.tokenize import sent_tokenize``` and then transformed each sentence to a vector in the embedded space with ```SentenceTransformer('all-MiniLM-L6-v2’)``` and compared ech sentence with the vector transformation of the definition provided in this [folder](Dict_of_terms) for each category of the theoretical framework: hegemonic state-centric, local turn, feminist security studies, and decolonial thought.
3. Then we computed the cosine similarity of the vector representation of each sentence in the abstract with the vector representation of each definition with ```similarity = np.dot(
              np.array(query_embedding), 
              np.array(sentence)
            ) / (
              np.linalg.norm(np.array(query_embedding)) * 
              np.linalg.norm(sentence)
            )
            ``` and selected the maximum value to represent each abstract.
4. Finally we labelled an article as related with a term if it contained  explicitly in the abstract or the primary topic or if cosine similarity of any sentence of the abstract was larger than 0.65: ```Filtered_DataFrame_out_filt[(Filtered_DataFrame_out_filt['Probability_term_'+concept].apply(lambda x: max(x)>=prob_passing)) | (Filtered_DataFrame_out_filt['Appearance_term_'+concept]=='Yes’)]```

You can fine the distribution of the cosine similarity for all terms in the [Cos_sim](Cos_sim) folder.

## Human verification <a name="humans">
To assess the accuracy of our analyses we performed two different human verifications:
1. We selected 5 abstracts for the top 5 languages with the most written abstracts (English, Spanish, Portuguese, Indonesian, and French) that were labelled as containing/related with three out of the 30 terms (Security, Feminist peace, and Decolonial peace), and 5 abstracts in the same language that were not labelled with the term. These terms were selected to have enough detected similar abstracts per language to conduct the analyses in the different paradigms analysed in the manuscript.  A total of 150 abstracts were checked by native speakers with knowledge in peace & conflict literature, 30 abstracts per language.

2. 

The code to generate the random samples and their verification after can be found in this [jupyter notebook](Multilingual_manual_verification_generate_check.ipynb)

## Citation analysis<a name="citations">

- **Maintainer:** Ana Maria Jaramillo (jaramillo@csh.ac.at)
- **License:** Apache 2.0
- **Contributors:** Ana Maria Jaramillo
