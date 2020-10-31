# Recipe_Workout plans recommender
A health-based recipe and exercise plan recommendation system.
A Line chatbot that recommend recipes based on user's ingredient preference, daily calories consumption, recipes style preference and other users preference behavior. Users can also send a photo of gym equipment, the chatbot will tell you what the equipment is and provide a link to a website that teaches you how to exercise with that equipment.


## Table of Content
**Data Collection**. 

**Data Preprocessing**. 

**Model Training**. 

**Data Pipeline**. 

**Linebot API**. 

**AWS Deployment**. 


## Data Collection
Websites we refered from:

* icook.com

* cookpad.com.tw

* allrecipes.com

* muscleandstrength.com

## Data Preprocessing

We use SparkSQL api to segment content in each recipe. 

1. Use jieba module for Chinese words segmentation. 
2. Define a Spark UDF that implementing jieba, regular expression for words segmentation
```
def wordToSeg(x):
	if not jieba.dt.initialized:
		#jieba.load_userdict('/home/spark/Desktop/recipe_com/mydict_3.txt')
		jieba.load_userdict('./mydict_3.txt')

	# Regular expression to eliminate non-chinese character
	try:
		interstate = re.sub(r'\W', '', x)
	except:
		interstate = x
		pass
	try:
		secondstate = interstate.replace('\n','')
	except:
		secondstate = interstate
		pass
	try:
		thirdstate = secondstate.replace('\n\n','')
	except:
		thirdstate = secondstate
		pass
	try:
		finalstate = re.sub(r'[a-zA-Z0-9]', '', thirdstate)
	except:
		finalstate = thirdstate
		pass
	try:
		seg = jieba.cut(finalstate, cut_all = False)
	except:
		output = finalstate
		pass
	try:
		output = ' '.join(seg)
	except:
		output = ''
		pass
	return output
```

## Model Training
1. Put all segmented words into Spark Mllib word2vec model training.

```
# Use PySpark SQL to preprocess dataset
recipe.createOrReplaceTempView("recipes")
recipes_seg = spark.sql('''select url, img_url, title, time, author, word2Seg(ingredient) ingredient, 
		word2Seg(steps) steps, word2Seg(comment) comment,
		word2Seg(category) category from recipes''')


recipes_seg.createOrReplaceTempView("recipes_seg")
recipes_wordbag = spark.sql('''SELECT concat(ingredient, steps, comment, category) as text from recipes_seg''')

......

# Split all text into list
recipes_wordbag.createOrReplaceTempView("recipes_wordlist")
for_word2vec = spark.sql('''SELECT word2list(text) text from recipes_wordlist''')

word2Vec = Word2Vec(vectorSize=50, minCount=3, inputCol="text", outputCol="result")
# Fit all words into word2vec model
model = word2Vec.fit(for_word2vec)
```
2. Then calculate words vector mean to represents each recipe.
3. Use cosine similarity to specify similar recipes.

## Data Pipeline & database structure

All services and databases are built in docker containers, including python devops environment, mongoDB, MySQL and kafka. 

1. Raw data collected from the web --> store in mongoDB.
2. Push data to Hadoop file system that runs on local machines.
3. Utilize SparkSQL to preprocess our datas, and Spark Mllib for model training.
4. Use Tensorflow for image recognition model training.
5. Build a Line Chatbot App in python devops docker container, with pipenv for libraries version control.
6. Construct docker-compose.yml file to run all containers.
7. Connect all containers by port mapping.

![alt text](https://github.com/asd855280/recipe_recommend_final/blob/master/demo_img/structure.png?raw=true)



## Line Chatbot API

1. Follow the chatbot.

![alt text](https://github.com/asd855280/recipe_recommend_final/blob/master/demo_img/follow.jpg)

2. Gym Equipment image recognition.

![alt text](https://github.com/asd855280/recipe_recommend_final/blob/master/demo_img/image_recog.jpg?raw=true)

3. Recipe recommendation.

![alt text](https://github.com/asd855280/recipe_recommend_final/blob/master/demo_img/recipe_recom.jpg?raw=true)

4. Functions as Saving workout plans or Saving recipes are also included. More demo images, please refer to demo_img directory.

## AWS Deployment
