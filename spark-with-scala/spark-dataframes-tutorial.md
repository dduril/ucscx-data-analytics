### Spark DataFrames Tutorial

#### Setup

	val folder = "/FileStore/tables/<your-folder-here>"
	val movie = spark.sparkContext.textFile(s"$folder/movies.csv")
	val movieRating = spark.sparkContext.textFile(s"$folder/movie_ratings-e9aa9.tsv")


#### Split Tab-Delimited

	val movieRDD = movie.map(line => line.split("\t"))
	val movieRatingRDD = movieRating.map(line => line.split("\t"))

#### Create Case Classes

	case class Movie(actor:String, title:String, year:Int)
	case class MovieRating(rating:String, title:String, year:Int)

#### Map Over Case Classes

	val movieRDD_ = movieRDD.map(m => Movie(m(0), m(1), m(2).toInt))
	val movieRatingRDD_ = movieRatingRDD.map(m => MovieRating(m(0), m(1), m(2).toInt))

#### Convert to DataFrames

val movieDF = movieRDD_.toDF
val movieRatingDF = movieRatingRDD_.toDF


#### Print Schemas and Observe Data

**movieDF**

	movieDF.printSchema
	
	root
	 |-- actor: string (nullable = true)
	 |-- title: string (nullable = true)
	 |-- year: integer (nullable = true)

	movieDF.show(10)
	(1) Spark Jobs
	+-----------------+--------------------+----+
	|            actor|               title|year|
	+-----------------+--------------------+----+
	|McClure, Marc (I)|       Freaky Friday|2003|
	|McClure, Marc (I)|        Coach Carter|2005|
	|McClure, Marc (I)|         Superman II|1980|
	|McClure, Marc (I)|           Apollo 13|1995|
	|McClure, Marc (I)|            Superman|1978|
	|McClure, Marc (I)|  Back to the Future|1985|
	|McClure, Marc (I)|Back to the Futur...|1990|
	|Cooper, Chris (I)|  Me, Myself & Irene|2000|
	|Cooper, Chris (I)|         October Sky|1999|
	|Cooper, Chris (I)|              Capote|2005|
	+-----------------+--------------------+----+
	only showing top 10 rows

**movieRatingDF**

	movieRatingDF.printSchema
	root
	 |-- rating: string (nullable = true)
	 |-- title: string (nullable = true)
	 |-- year: integer (nullable = true)
	
	movieRatingDF.show(10)
	(1) Spark Jobs
	+-------+--------------------+----+
	| rating|               title|year|
	+-------+--------------------+----+
	| 1.6339|'Crocodile' Dunde...|1988|
	| 7.6177|                  10|1979|
	| 1.2864|10 Things I Hate ...|1999|
	| 0.3243|           10,000 BC|2008|
	| 0.3376|      101 Dalmatians|1996|
	| 0.5218|      102 Dalmatians|2000|
	|12.8205|                1066|2012|
	| 0.6829|                  12|2007|
	| 7.4061|           12 Rounds|2009|
	| 2.3677|           127 Hours|2010|
	+-------+--------------------+----+
	only showing top 10 rows

#### Begin Join Process

Drop one of the 'year' columns and join dataframes on 'title'.

	val movieRatingDF_ = movieRatingDF.drop("year")
	val movie_movieRatingDF = movieDF.join(movieRatingDF_, "title")
	
	movie_movieRatingDF.printSchema
	root
	 |-- title: string (nullable = true)
	 |-- actor: string (nullable = true)
	 |-- year: integer (nullable = true)
	 |-- rating: string (nullable = true)
	
	movie_movieRatingDF.show(10)
	(1) Spark Jobs
	+-----------------+--------------------+----+------+
	|            title|               actor|year|rating|
	+-----------------+--------------------+----+------+
	|Failure to Launch|     Caudle, Melissa|2006|0.6775|
	|Failure to Launch|    Deschanel, Zooey|2006|0.6775|
	|Failure to Launch|   Winnick, Katheryn|2006|0.6775|
	|Failure to Launch|          Cool, Greg|2006|0.6775|
	|Failure to Launch|      Bartha, Justin|2006|0.6775|
	|Failure to Launch|    Chalaire, Kristi|2006|0.6775|
	|Failure to Launch|McGregor-Stewart,...|2006|0.6775|
	|Failure to Launch|      Tovah, Mageina|2006|0.6775|
	|Failure to Launch|        Corddry, Rob|2006|0.6775|
	|Failure to Launch|McConaughey, Matthew|2006|0.6775|
	+-----------------+--------------------+----+------+
	only showing top 10 rows
	
Select title, rating, and year and order by title.

	val movieListDF = movie_movieRatingDF
		.select('title, 'rating, 'year)
		.distinct()
		.orderBy('title)
		.show(20)
	
	(1) Spark Jobs
	+--------------------+------+----+
	|               title|rating|year|
	+--------------------+------+----+
	|'Crocodile' Dunde...|1.6339|1988|
	|10 Things I Hate ...|1.2864|1999|
	|           10,000 BC|0.3243|2008|
	|      101 Dalmatians|0.3376|1996|
	|      102 Dalmatians|0.5218|2000|
	|                  12|0.6829|2007|
	|      13 Going on 30|1.3585|2004|
	|                1408|  0.59|2007|
	|            17 Again|1.0491|2009|
	|    2 Fast 2 Furious|   0.4|2003|
	|                  21|1.0392|2008|
	|      21 Jump Street|0.5352|2012|
	|          27 Dresses|0.5407|2008|
	|             28 Days|1.4889|2000|
	|    28 Days Later...|1.0344|2002|
	|                 300|0.1918|2006|
	|        3:10 to Yuma|1.3744|2007|
	|40 Days and 40 Ni...|1.9327|2002|
	|4: Rise of the Si...|0.2991|2007|
	|      50 First Dates|0.3808|2004|
	+--------------------+------+----+
	only showing top 20 rows
	
Select title and year, order by year, and count number of movies for each year.

	val moviesPerYearDF = movieDF.select('title, 'year)
		.distinct()
		.orderBy($"year".desc)
		.groupBy('year)
		.count()
		.show(50)

	(5) Spark Jobs
	+----+-----+
	|year|count|
	+----+-----+
	|2012|   32|
	|2011|   86|
	|2010|   78|
	|2009|   68|
	|2008|   82|
	|2007|   75|
	|2006|   86|
	|2005|   85|
	|2004|   86|
	|2003|   76|
	|2002|   81|
	|2001|   71|
	|2000|   77|
	|1999|   67|
	|1998|   59|
	|1997|   66|
	|1996|   42|
	|1995|   25|