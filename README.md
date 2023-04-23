# KnowledgeBaseFromText

## Code description
The main task is build a knowledge base from text (or from an online article).
To do that we need to extract from a text three class of information:
1. Entities
2. Relation
3. Sources

We use an end-to-end model called REBEL that allow us to tackle the tasks simultaneously. 

>>>**REBEL is a text2text model trained by BabelScape by fine-tuning BART for translating a raw input sentence containing entities and implicit relations into a set of triplets that explicitly refer to those relations.**


## Implementation
After the installation of the libraries we need to import them in the code. Especially the libraries and the reasons we need them are:

* transformers: Load the REBEL mode.
* wikipedia: Validate extracted entities by checking if they *have a corresponding Wikipedia page.
* newspaper: Parse articles from URLs.
* GoogleNews: Read Google News latest articles about a topic.
* pyvis: Graphs visualizations.


## Description of the code lines

### *extract_relations_from_model_output(text)*
This function takes a text string as input and returns a list of relation triplets. First of all the function need to edit the text to prepare it for the analysis. 

* **text = text.strip ()** 
The strip() method returns a copy of the string by removing both the leading and the trailing characters (based on the string argument passed)

* **text_replaced = text.replace ("< s >"," ").repla... ecc...**
This line replaces three special markers in the input text string with an empty string. This replacements are necessary because the function expect to receive a string without this markers


After that the function can finally split the text_replaced in differents tokens and analyze them one by one, creating the relation triplets and adding them in the relation list



### *class KB()*
this class KB need to have two different list of objects, one for the entities and one for relations. In the class there are the implementation of functions used for the management of the KB.

1. are_relations_equal(self, r1, r2): This function takes two relation r1 and r2 as inputs and checks if they are equal. It does so by iterating through the common attributes of r1 and r2 (i.e., head, type, and tail) and checking if they have the same value.
2. exists_relation(self, r1): This function takes a relation r1 as input and checks if it already exists in the knowledge base's relations list. It does so by iterating through the relations list and checking if r1 is equal to any of the existing relations using the are_relations_equal function.
3. merge_relations(self, r1): This function takes a relation r1 as input and merges it with an existing relation in the knowledge base's relations list. It does so by finding the existing relation that is equal to r1 using the are_relations_equal function, and adding any new spans (i.e., mentions of the same relation in text) to the existing relation's metadata.
4. merge_with_kb(self,kb2): This function allows you to combine two KB instances into one, while preserving the article metadata associated with each relation. The method takes in one parameter kb2, which is the KB instance to be merged. For each relation in kb2, it extracts the article_url key from the meta dictionary, and then retrieves the corresponding article metadata (i.e., title and publish date) from the sources dictionary of kb2. It then adds the relation to the current KB instance by calling the add_relation() method and passing in the relation, article title, and publish date as arguments.
5. get_wikipedia_data(self, candidate_entity): This function takes a candidate entity as input and retrieves its data from Wikipedia using the wikipedia package. It returns a dictionary containing the entity's title, URL, and summary. If the entity is not found on Wikipedia, it returns None.
6. add_entity(self, e): This function takes an entity dictionary e as input and adds it to the knowledge base's entities dictionary, using the entity's title as the key and a dictionary of its other attributes as the value.
7. add_relation(self, r): This function takes a relation r as input and adds it to the knowledge base's relations list. It first checks if the relation's head and tail entities exist on Wikipedia using the get_wikipedia_data function. If either entity is not found, the relation is not added to the knowledge base. If both entities exist, they are added to the knowledge base's entities dictionary using the add_entity function. Then, the relation's head and tail attributes are updated with their respective Wikipedia titles. Finally, if the relation does not already exist in the relations list, it is added. If it does exist, it is merged with the existing relation using the merge_relations function.
8. print(self): This function prints the entities and relations in the knowledge base in a readable format. It iterates through the entities dictionary and relations list and prints each entity's title and attributes, and each relation's head, type, tail, and metadata.



### *from_text_to_kb(text, article_url,...)*
This function takes a text as input and returns a KB objects. The purpose of this function is to extract relations from the input text and store them in the KB object.

How the funtion do that?
* Tokenizes the text using a pre-trained tokenizer
* Divides the tokenized text into spans of a given length
* Generates relations from each span of the text using a pre-trained language model
* Decode the generated relations to obtain the predicted relations
* Creates a KB object and adds the predicted relation to it

>>>**In the previous version there was another type of function like that but called 'from_small_text_to_kb()' that worked with a small text and for that it wasn't necessary dividing the text into spans. Transformer models like REBEL have memory requirements that grow quadratically with the size of the inputs. This means that REBEL is able to work on common hardware at a reasonable speed with inputs of about 512 tokens, which correspond to about 380 English words. However, we may need to extract relations from documents long several thousands of words. Moreover, It seems to work better with shorter inputs. Intuitively, raw text relations are often expressed in single or contiguous sentences, therefore it may not be necessary to consider a high number of sentences at the same time to extract specific relations. Additionally, extracting a few relations is a simpler task than extracting many relations**



### *get_article(url)*
this function takes a URL as an input and returns the full text article from that URL using the Article class from the newspaper library. It downloads the article content and extracts the text using article.parse() method, and returns the resulting article object.

### *from_url_to_kb(url)* 
This function takes a URL as an input and returns a knowledge base (KB) generated from the full text article. It first calls the get_article(url) function to retrieve the article content from the URL, and then it passes that text to the from_text_to_kb function along with additional parameters such as article_title and article_publish_date. Finally, it returns the resulting KB object.

### *get_news_links(query, lang="en",...)*
This function takes a search query and returns a list of news article links related to the query. It uses the GoogleNews module to perform the search and get the links. The function has several parameters:

* query: the search query to look up in the news. We will use 'Google'.
* lang: the language in which the articles are written. The default value is "en" for English.
* region: the region where the articles are published. The default value is "US" for United States.
* pages: the number of pages to retrieve from the search results. The default value is 1.
* max_links: the maximum number of links to return. If the number of links retrieved is greater than this value, the function will return only the first max_links links. The default value is 100000.

### *from_urls_to_kb(urls, verbose=False)*
This function takes in a list of URLs and returns a KB (knowledge base) object. The function iterates through the list of URLs, and for each URL, it calls the from_url_to_kb function to download the article and convert it to a KB object. It then merges the KB object obtained from the URL with the main KB object. If an ArticleException is raised during the download process, the function skips that URL and moves on to the next one. Finally, the function returns the merged KB object.



# TESTS

There are three tests to understand what the code produce 

### 1. CREATION OF THE KB FROM A TEXT
first of all we need to test if the code is ready to create a KB from a normal text. The text isn't taked by a file but we inizialize it as a string variable. Next the other lines are: 

* **kb = from_text_to_kb(text, None)**
* **kb.print()**

That's all, the first line create the KB and the second one allow to visualize the result


### 2. CREATION OF THE KB FROM AN ONLINE ARTICLE
Once checked that the first test is successful we can continue to check the second one. The difference between the previous method is that now we first need to extract the text from an URL. In this case the URL is written in a string to speed up the process. 

Sn this case we write this line: 
* **url = "https://finance.yahoo.com/news/microstrategy-bitcoin-millions-142143795.html"**

And the other lines are: 

* **kb2 = from_url_to_kb(url)**
* **kb2.print()**

That's all, the first line create the KB and the second one allow to visualize the result

### 3. CREATION OF THE KB FROM DIFFERENT ONLINE ARTICLES
That's the third test. In this case we need different articles but we don't write them as an string inizializzation but we use the function *get_new_links* that returns a list of news articles.

the lines are:
* **news_links = get_news_links("Google", pages=1, max_links=3)**
* **kb3 = from_urls_to_kb(news_links, verbose=True)**
* **kb3.print()**

The second one create the KB and the third allow us to visualize the result