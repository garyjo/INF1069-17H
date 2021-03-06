# Les opérations CRUD

## Opérations atomic

Une opération *atomic* est une combinaison de plusieurs opérations 
qui sont exécutées comme une seule opération - cad, soit l'opération réussit ou elle échoue.
On parle d'opération *atomic*, si:

* Aucun processus n'est au courant du changement jusqu'à ce que le changement soit complété.
* Si une opération échoue, l'ensemble d'opérations est annulé (rollback).

## Règles sur la nomenclature d'une collection

* Pas plus que 128 caractères.
* Ne peut pas avoir un champ de caractère vide (" ") comme nom.
* Doit commencer par une lettre ou bien underscore _ 
*Exemple: 6019_inf est invalide*
* Ne peut pas utiliser le mot *system* pour nom, car c'est une collection reservée par MongoDB.
* Ne peut pas contenir le caractère NULL (code ascii est "\0").

## Règles à considérer pour un document

* Le caractère $ ne peut pas être le 1er caractère dans un nom d'un champ. *Exemple: $tags*
* Le caractère [.] ne doit pas être utilisé dans le nom de la clé. *Exemple: ta.gs*
* Le champ *_id* est réservé pour la clé primaire.
* Bien que ce n'est pas recommandé, le *_id* peut contenir une chaine de caractère ou un integer.

## La fonction INSÉRER

### Syntaxe

```
db.collection.insert(<document or array of documents>,
					{writeConcern: <document>,ordered:<boolean>})
```

### En utilisant une variable

```
> use semaine02
> db.catalog.drop()
> doc1 = {
			"catalogId" : 1, 
			"journal" : 'Oracle Magazine',
			"publisher" : 'Oracle Publishing',
			"edition" : 'November December 2013',
			"title" : 'Engineering as a Service',
			"author": 'David A. Kelly'
		}
> db.catalog.insert(doc1)
> db.catalog.find().pretty()
```

Note: La collection est créée automatiquement si elle n'existe pas.
On n'est pas obligé de la créer explicitement.

```
> document = ({ 
				"Type" : "Book",
				"Title" : "Definitive Guide to MongoDB 2nd ed., The",
				"ISBN" : "978-1-4302-5821-6",
				"Publisher" : "Apress",
				"Author": [
							"Hows, David",
							"Plugge, Eelco",
							"Membrey, Peter",
							"Hawkins, Tim" 
						] 
				} )
> db.media.insert(document)
> db.media.find().pretty()
```

### Directement dans la ligne de commande sans définir de variable

```
> db.media.insert( { "Type" : "CD", "Artist" : "Nirvana", "Title" : "Nevermind" })
> db.media.find().pretty()
```

### Directement dans la ligne de commande avec des sauts de lignes

```
> db.media.insert( { "Type" : "CD",
"Artist" : "Nirvana",
"Title" : "Nevermind",
"Tracklist" : 
	[
		{
			"Track" : 1,
			"Title" : "Smells Like Teen Spirit",
			"Length" : "5:02"
		},
		{
			"Track" : "2",
			"Title" : "In Bloom",
			"Length" : "4:15"
		}
	]
})
> db.media.find().pretty()
```

### En spécifiant implicitement une valeur pour le champ *_id*

```
> doc1 = {
			"_id": ObjectId("507f191e810c19729de860ea"),
			"catalogId" : 1,
			"journal": 'Oracle Magazine',
			"publisher" : 'Oracle Publishing',
			"edition" : 'November December 2013',
			"title" : 'Engineering as a Service',
			"author" : 'David A. Kelly'
		}
> db.catalog.insert(doc1)
> db.catalog.find().pretty()
```

### La clé *_id* doit être unique

```
> doc2 = {
			"_id": ObjectId("507f191e810c19729de860ea"),
			"catalogId" : 2,
			"journal" : 'Oracle Magazine',
			"publisher" : 'Oracle Publishing',
			"edition" : 'November December 2013'
		}
> db.catalog.insert(doc2)
> doc3 = {"_id": ObjectId("507f191e810c19729de860ea"),"catalogId" : 3}
> db.catalog.insert(doc3)
> db.catalog.find().pretty()
```

### Insérer plusieurs documents

```
> doc1 = {
			"_id": ObjectId("507f191e810c19729de860ea"),
			"catalogId" : 1,
			"journal" : 'Oracle Magazine',
			"publisher" : 'Oracle Publishing',
			"edition" : 'November December 2013',
			"title": 'Engineering as a Service',
			"author" : 'David A. Kelly'
		}
> doc2 = {
			"_id" : ObjectId("53fb4b08d17e68cd481295d5"),
			"catalogId" : 2,
			"journal" : 'Oracle Magazine',
			"publisher" : 'Oracle Publishing',
			"edition" : 'November December 2013',
			"title": 'Quintessential and Collaborative',
			"author" : 'Tom Haunert'
		}
> db.catalog.drop()
> show collections
> db.catalog.insert([doc1, doc2])
> db.catalog.find().pretty()
```

Autre exemple

```
> doc1 = {
			"_id": ObjectId("507f191e810c19729de860ea"),
			"catalogId" : 2,
			"journal" : 'Oracle Magazine',
			"publisher" : 'Oracle Publishing', 
			"edition" : 'November December 2013',
			"title" : 'Engineering as a Service',
			"author" : 'David A. Kelly'
		}
> doc2 = {
			"_id" : ObjectId("53fb4b08d17e68cd481295d5"), 
			"catalogId" : 1, 
			"journal" : 'Oracle Magazine', 
			"publisher" : 'Oracle Publishing', 
			"edition" : 'November December 2013',
			"title": 'Quintessential and Collaborative',
			"author" : 'Tom Haunert'
		}
> doc3 = {
			"_id" : ObjectId("53fb4b08d17e68cd481295d6"),
			"catalogId" : 3, 
			"journal" : 'Oracle Magazine', 
			"publisher" : 'Oracle Publishing', 
			"edition" : 'November December 2013'
		}
> db.catalog.drop()
> show collections
> db.catalog.insert([doc3, doc1, doc2], { writeConcern: { w: "majority", wtimeout: 5000 }, ordered:true })
```

## La fonction MODIFIER

### Syntaxe

```
db.collection.update(query, update, options)
db.collection.update(
	<query>,
	<update>,
	{
		upsert: <boolean>,
		multi: <boolean>,
		writeConcern: <document>
	}
)
```

* La fonction prend 3 paramètres: *criteria, objNew et options*.
* Le paramètre *criteria* permet de spécifier le critère pour retrouver la donnée à modifier.
* Le paramètre *objNew* permet de spécifier la donnée mise à jour; 
* ou sinon utiliser un opérateur pour le faire. 
* Le paramètre *options* permet de spécifier les options et les values possibles sont: *upsert* et *multi*.
** upsert: mettre à jour ou créer
** multi: tous les documents trouvés ou la 1er seulement.

### Insérer dans notre catalogue en premier, puis modifier

```
> db.catalog.drop()
> show collections
> doc1 = {
			"catalogId" : 1, 
			"journal" : 'Oracle Magazine', 
			"publisher": 'Oracle Publishing', 
			"edition" : 'November December 2013'
		}
> db.catalog.insert(doc1)
> db.catalog.find().pretty()
```

Modifier le document

```
> db.catalog.update(
	{ 
		catalogId: 1
	},
	{
		"catalogId" :  1,
		"journal" : 'Oracle Magazine', 
		"publisher" : 'Oracle Publishing', 
		"edition" : '11122013',
		"title" : 'Engineering as a Service',
		"author": 'Kelly, David A.'
	}
)
> db.catalog.find().pretty()
```

### Pas besoin d'insérer, *upsert: true*

```
> db.media.update( 
	{ "Title" : "Matrix, The"}, 
	{
		"Type" : "DVD", 
		"Title" : "Matrix, The", 
		"Released" : 1999, 
		"Genre" : "Action"
	}, 
	{ upsert: true} 
)
> db.media.find({"Title" : "Matrix, The"}).pretty()
```

### Utiliser *upsert* et *multi* quand on utilise l'opérateur *$set*

```
> db.media.update( 
	{ "Title" : "Matrix, The"}, 
	{	
		$set: {
				"Type" : "Blue-R", 
				"Title" :"Matrix, The", 
				"Released" : 1999, 
				"Genre" : "Action",
				"Published": 2017
			} 
	}, 
	{upsert: true, multi: true} 
)
> db.media.find().pretty()

> db.catalog.update(
	{ journal: 'Oracle Magazine'},
	{
		$set: { edition: '11129999'},
		$inc: { catalogId: 2 }
	},
	{
		multi: true,
		writeConcern: { w: 1, wtimeout: 5000 }
	}
)
> db.catalog.find().pretty()
```

### Modifier plusieurs document à la fois

```
> use semaine02
> db.catalog.drop()
> show collections
> doc1 = {
			"catalogId" : 1, 
			"journal" : 'Oracle Magazine', 
			"publisher" : 'Oracle Publishing', 
			"edition" : 'November December 2013',
			"title" : 'Engineering as a Service',
			"author" : 'David A. Kelly'
		}
> db.catalog.insert(doc1)
> doc2 = {
			"catalogId" : 2, 
			"journal" : 'Oracle Magazine', 
			"publisher" : 'Oracle Publishing', 
			"edition" : 'November December 2013',
			"title" : 'Quintessential and Collaborative',
			"author" : 'Tom Haunert'
		}
> db.catalog.insert(doc2)
> db.catalog.find().pretty()
```

Ensuite, modifier

```
> db.catalog.update(
   { journal: 'Oracle Magazine'},
   {
      $set: { edition: '11­12­2013' },
      $inc: { catalogId: 2 }
   },
   {
     multi: true,
     writeConcern: { w: 1, wtimeout: 5000 }
   }
)
> db.catalog.find({ journal: 'Oracle Magazine'}).pretty()
```

### La fonction *save()*

#### Syntaxe

```
db.collection.save(<document>,{writeConcern: <document>})
```

En utilisant *save*, il faut spécifier le *_id*, sinon il fait un *insert*.
La commande *save* vous permet de simplifier la syntaxe. Le résultat est le même.
Exemple: On insert avant

```
> use semaine02
> db.catalog.drop()
> doc1 = {
			"_id": ObjectId("507f191e810c19729de860ea"), 
			"catalogId" : 1,
			"journal" : 'Oracle Magazine', 
			"publisher" : 'Oracle Publishing',
			"edition": 'November December 2013',
			"title" : 'Engineering as a Service',
			"author" : 'David A. Kelly'
		}
> db.catalog.insert(doc1)
> db.catalog.find().pretty()
```

#### Construire un document avec le même *_id*

```
> doc1 = {
			"_id": ObjectId("507f191e810c19729de860ea"), 
			"catalogId" : 1, 
			"journal" : 'Oracle Magazine', 
			"publisher" : 'Oracle Publishing', 
			"edition" : '11122013',
			"title" : 'Engineering as a Service',
			"author" : 'Kelly, David A.'
		}
> db.catalog.save(doc1,{ writeConcern: { w: "majority", wtimeout: 5000 } })
> db.catalog.find().pretty()
```

#### Modifier la structure d'un document existant

```
doc2 = {
			"_id": ObjectId("507f191e810c19729de860ea"), 
			"catalogId" : 1,
			"journal" : 'Oracle Magazine', 
			"publisher" : 'Oracle Publishing', 
			"edition" : 'November December 2013'
		}
> db.catalog.drop()
> db.catalog.insert(doc2)
> db.catalog.find().pretty()
```

Ajout des champs title et author

```
> doc2 = {
			"_id": ObjectId("507f191e810c19729de860ea"),
			"catalogId" : 2,
			"journal" : 'Oracle Magazine', 
			"publisher" : 'Oracle Publishing', 
			"edition" : '11122013',
			"title" : 'Quintessential and Collaborative',
			"author" : 'Tom Haunert'
		}
> db.catalog.save(doc2)
> db.catalog.find().pretty()
```

#### Modifier ou ajouter si le document n'existe pas

```
> db.catalog.drop()
> doc2 = {
			"_id": ObjectId("507f191e810c19729de860ea"),
			"catalogId" : 2,
			"journal" : 'Oracle Magazine', 
			"publisher" : 'Oracle Publishing', 
			"edition": 'November December 2013'
		}
> db.catalog.insert(doc2)
> db.catalog.find().pretty()

> doc2 = {
			"_id": ObjectId("507f191e810c19729de860eb"),
			"catalogId" : 3,
			"journal" : 'Oracle Magazine', 
			"publisher" : 'Oracle Publishing', 
			"edition" : '11122013',
			"title" : 'Quintessential and Collaborative',
			"author" : 'Tom Haunert'
		}
> db.catalog.save(doc2)
> db.catalog.find().pretty()
```

#### Sans préciser le *_id*

```
> db.catalog.drop()
> doc2 = {
			"catalogId" : 2, 
			"journal" : 'Oracle Magazine', 
			"publisher": 'Oracle Publishing', 
			"edition" : 'November December 2013'
		}
> db.catalog.insert(doc2)
> db.catalog.find().pretty()
```

Ajout des champs title et author

```
> doc2 = {
			"catalogId" : 3, 
			"journal" : 'Oracle Magazine', 
			"publisher": 'Oracle Publishing', 
			"edition" : '11122013',
			"title" : 'Quintessential and Collaborative',
			"author" : 'Tom Haunert'
		}
> db.catalog.save(doc2)
> db.catalog.find().pretty()
```

#### update vs save

Update

```
> db.media.update( 
	{ 
		"Title" : "Matrix, The"
	}, 
	{
		"Type" : "DVD", 
		"Title" : "Matrix, The", 
		"Released" : "1999", 
		"Genre" : "Action"
	}, 
	{ upsert: true} 
)
> db.media.find().pretty()
```

Save

```
> db.media.save( 
	{ 
		"Title" : "Matrix, The"
	}, 
	{
		"Type" : "DVD", 
		"Title" : "Matrix, The", 
		"Released" : "1999", 
		"Genre" : "Drama"
	}
)
> db.media.find({"Title" : "Matrix, The"}).pretty()
```

### Incrémenter une valeur avec l'opérateur *$inc*

* Performe une opération *atomic*.
* Si la champ n'existe pas, il sera créé.

```
> manga = ( 
	{ 
		"Type" : "Manga", 
		"Title" : "One Piece", 
		"Volumes" : 612,
		"Read" : 520 
	} 
)
> db.media.insert(manga)
> db.media.find({"Type" : "Manga"}).pretty()

> db.media.update ( { "Title" : "One Piece"}, {$inc: {"Read" : 4} } )
> db.media.find ( { "Title" : "One Piece" } ).pretty()
```

### Opérateur *$set*

Change la valeur, sinon crée la champ avec la valeur.

```
> db.media.update ( 
	{ 
		"Title" : "Matrix, The" 
	},
	{
		$set : { Genre :"Sci-Fi" } 
	} 
)
> db.media.find ( { "Title" : "Matrix, The" } ).pretty()
```

### Opérateur *$unset*

Supprime le champ.

```
> db.media.update ( {"Title": "Matrix, The"}, {$unset : { "Genre" : 1 } } )
> db.media.find ( { "Title" : "Matrix, The" } ).pretty()
```

### Opérateur *$push*

Permet d'ajouter à la fin, *append*. Ne s'applique qu'à un tableau.

```
> db.media.update ( {"ISBN" : "978-1-4302-5821-6"}, {$push: { Author : "Griffin, Stewie"} } )
> db.media.find ( { "ISBN" : "978-1-4302-5821-6" } ).pretty()
```

Ne s'applique qu'à un tableau

```
> db.media.update ( {"ISBN" : "978-1-4302-5821-6"}, {$push: { Title : "This isn't an array"} } )
Cannot apply $push/$pushAll modifier to non-array
```

### Spécifier plusieurs valeurs pour un tableau

```
> db.media.update( 
	{ 
		"ISBN" : "978-1-4302-5821-6" 
	}, 
	{ 
		$push: 
		{ 
			Author : 
			{ 
				$each: ["Griffin, Peter", "Griffin, Brian"] 
			} 
		} 
	} 
)
> db.media.find ( { "ISBN" : "978-1-4302-5821-6" } ).pretty()
```

Utiliser *$slice* pour limiter la taille du tableau lors du *push* - c'est optionnel
Prend une valeur négative ou 0.

```
> db.media.update( 
	{ 
		"ISBN" : "978-1-4302-5821-6" 
	}, 
	{ 
		$push: 
		{ 
			Author : 
			{ 
				$each: ["Griffin, Meg", "Griffin, Louis"], 
				$slice: -2 
			} 
		} 
	} 
)
> db.media.find ( { "ISBN" : "978-1-4302-5821-6" } ).pretty()
```

### Ajouter des données dans un tableau avec *$addToSet*

La différence avec *$push*, il n'ajoute qu'une valeur qui n'est pas présente.

```
> db.media.update( 
	{ 
		"ISBN" : "1-4302-3051-7" 
	}, 
	{
		$addToSet : { Author : "Griffin, Brian" } 
	} ,
	{ upsert: true}
)
> db.media.find ( { "ISBN" : "1-4302-3051-7" } ).pretty()

> db.media.update( 
	{ 
		"ISBN" : "1-4302-3051-7" 
	}, 
	{
		$addToSet : 
		{ 
			Author : { $each : ["Griffin, Brian","Griffin, Meg"] } 
		} 
	} 
)
> db.media.find ( { "ISBN" : "1-4302-3051-7" } ).pretty()
```

### Supprimer des éléments dans un tableau

#### *$pop*, supprime un seul élément selon le critère

```
> db.media.update( { "ISBN" : "1-4302-3051-7" }, {$pop : {Author : 1 } } )
> db.media.find ( { "ISBN" : "1-4302-3051-7" } ).pretty()
> db.media.update( { "ISBN" : "1-4302-3051-7" }, {$pop : {Author : -1 } } )
> db.media.find ( { "ISBN" : "1-4302-3051-7" } ).pretty()
```

#### *$pull*, supprime chaque occurence, selon le critère

```
> db.media.update ( 
	{
		"ISBN" : "1-4302-3051-7"
	}, 
	{
		$pull: { Author : "Griffin, James" } 
	} 
)
> db.media.find ( { "ISBN" : "1-4302-3051-7" } ).pretty()

> db.media.update ( 
	{
		"ISBN" : "1-4302-3051-7"
	}, 
	{
		$pull : { Author : "Griffin, Stewie" } 
	} 
)
> db.media.find ( { "ISBN" : "1-4302-3051-7" } ).pretty()
```

#### *$pullAll*, supprime tous les éléments selon le critère

```
> db.media.update( 
	{ 
		"ISBN" : "1-4302-3051-7"
	}, 
	{
		$pullAll : 
		{ 
			Author : ["Griffin, Louis","Griffin, Peter","Griffin, Brian"] 
		} 
	} 
)
> db.media.find ( { "ISBN" : "1-4302-3051-7" } ).pretty()
```

### Préciser une position dans le tableau

```
> db.media.update( 
	{ 
		"Artist" : "Nirvana" 
	}, 
	{
		$addToSet : 
		{ 
			Tracklist : {"Track" : 2,"Title": "Been a Son", "Length":"2:23"} 
		} 
	} 
)
> db.media.find ( { "Artist" : "Nirvana" } ).pretty()
```

### La fonction *findAndModify*

* Prend 3 paramètres query, sort, operations
* Utilise tous les opérateurs de modification sauf $set

#### Syntaxe

```
db.collection.findAndModify({
	query: <document>,
	sort: <document>,
	remove: <boolean>,
	update: <document>,
	new: <boolean>,
	fields: <document>,
	upsert: <boolean>
})
```

#### Modifier, trier, upsert

```
> db.catalog.findAndModify({
	query: {journal : "Oracle Magazine"},
	sort: {catalogId : 1},
	update: {$inc: {catalogId: 3}, $set: {edition: '11122013'}},
	upsert :true,
	new: true,
	fields: {catalogId: 1, edition: 1, title: 1, author: 1}
})
> db.getCollection('catalog').find({}).pretty()
```

#### Exception, ne pas combiner *upsert* avec *remove*

```
> db.catalog.findAndModify({
	query: {journal : "Oracle Magazine"},
	sort: {catalogId : 1},
	update: {$inc: {catalogId: 1}, $set: {edition: '11122013'}},
	remove: true,
	upsert :true,
	new: true,
	fields: {catalogId: 1, edition: 1, title: 1, author: 1}
})
```

## La fonction SUPPRIMER

### Syntaxe

```
db.collection.remove(
	<query>,
	{
		justOne: <boolean>,
		writeConcern: <document>
	}
)
```

### Bonne pratique

C'est recommandé de faire un find() pour retrouver la donnée en premier avant de supprimer.

### Supprimer un document spécifique

Supprime toutes les documents ayant ce Title.

```
> db.catalog.drop()
> doc1 = {
			"_id" : ObjectId("53fb4b08d17e68cd481295d5"),
			"catalogId" : 1, 
			"journal" : 'Oracle Magazine',
			"publisher" : 'Oracle Publishing', 
			"edition" : 'November December 2013',
			"title": 'Engineering as a Service',
			"author" : 'David A. Kelly'
		}
> db.catalog.insert(doc1)
> db.catalog.find().pretty()

> doc2 = {
			"_id" : ObjectId("53fb4b08d17e68cd481295d6"), 
			"catalogId" : 2, 
			"journal" : 'Oracle Magazine',
			"publisher" : 'Oracle Publishing', 
			"edition" : 'November December 2013',
			"title": 'Quintessential and Collaborative',
			"author" : 'Tom Haunert'
		}
> db.catalog.insert(doc2)
> db.catalog.find().pretty()

> doc3 = {
			"_id" : ObjectId("53fb4b08d17e68cd481295d7"), 
			"catalogId" : 3, 
			"journal" : 'Oracle Magazine',
			"publisher" : 'Oracle Publishing', 
			"edition" : 'November December 2013'
		}
> db.catalog.insert(doc3)
> db.catalog.remove({ catalogId: 2 })
> db.catalog.find().pretty()
```

### Autres exemples avec le paramètre *justOne*

```
> db.catalog.remove(
	{ catalogId: { $gt: 1 } },
	{ justOne:true, writeConcern: { w: 1, wtimeout: 5000 } }
)
> db.catalog.find().pretty()
```

### Supprime tous les documents dans cette collection

```
> db.media.remove({})
> db.catalog.remove({})
```

### Supprimer la collection et les documents

```
> db.media.drop()
true
> db.catalog.drop()
true
```

### Particularité d'une collection fixe, *capped*

* On ne peut pas supprimer des documents.
* Il faut faire *drop* et recréer.
* Il supprime la plus ancienne valeur et insère de façon circulaire.

```
> db.createCollection("catalog", {capped: true, size: 64 * 1024, max: 1000} )
```

### Supprimer la base de données courante

```
> db.dropDatabase()
{ "dropped" : "library", "ok" : 1 }
```
