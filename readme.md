# BigData Exam -- Emmanuel KOUPOH

1. Créer votre base Mongo DB.

        use exam

1. Importer ces deux fichiers Films et Utilisateurs sur Mongo Db.

        mongoimport --db exam --collection Films --type json --file Films.json "mongodb+srv://cluster0.vlhcs.mongodb.net/" --username koupoh

   ...

        mongoimport --db exam --collection Utilisateurs --type json --file Utilisateurs.json "mongodb+srv://cluster0.vlhcs.mongodb.net/" --username koupoh

1. Retourner le nombre d’utilisateurs dans la base de données.

        db.Utilisateurs.countDocuments()

1. Donner le nombre de films dans la base de données.

        db.Films.countDocuments()

1. Donner l’occupation de Shawn Saul.La requête affiche uniquement son nom et son occupation.

        db.Utilisateurs.find({ nom: "Shawn Saul" }, { _id: 0, nom: 1, occupation: 1, })

1. Donner le nombre d’utilisateurs dont l’age est entre 18 et 30 ans.

        db.Utilisateurs.countDocuments({ age: { $gt: 18, $lt: 30 } })

1. Donner le nombre utilisateurs programmeurs(programmer) ou écrivains(writer).

        db.Utilisateurs.countDocuments(
            {
                $or: [
                    { occupation: "programmer" },
                    { occupation: "writer" }]
            })

1. Afficher les 3 hommes artistes(artist) les plus âgées.

        db.Utilisateurs.find(
            {
                genre: "M",
                occupation: "artist"
            },
            //{nom:1, age:1} // pour avoir juste le nom, l'age et l'id
        ).limit(3).sort({ age: -1 })

1. Afficher les différentes occupations présentes dans la base de données.

        db.Utilisateurs.distinct("occupation")

1. Insérer un nouvel utilisateur dans la base de données.Ne pas mettre pour l’instant le champ films.

        db.Utilisateurs.insertOne(
            {
                "nom": "Emmanuel Koupoh",
                "genre": "M",
                "age": 1,
                "occupation": "programmer",
            })

1. Choisir un film de la collection films et mettre à jour l’enregistrement ajoutée à la question précédente en ajoutant le champ films respectant le schéma adopté par les autres enregistrements.Pour le champ timestamp, utiliser l’heure courante: Math.round(new Date().getTime() / 1000)

        db.Utilisateurs.findOneAndUpdate(
            { "nom": { $regex: "Emmanuel Koupoh" } },
            {
                "$set":
                {
                    "films": [{
                        "idfilm": db.Films.findOne({ "titre": { $regex: "mortal" } }, { _id: 1 })["_id"],
                        "note": 4, 
                        "timestamp": Math.round(new Date().getTime() / 1000)
                    }]
                }
            })

1. Supprimer l’enregistrement de la base de données.

        db.Utilisateurs.deleteOne({ "nom": { $regex: "Emmanuel Koupoh" } })

1. Changer l’occupation de tous les utilisateurs "programmer" en la remplaçant par "developer".

        db.Utilisateurs.updateMany({ occupation: "programmer" },
            [
                { "$set": { "occupation": "developer" } }
            ])

1. Donner le nombre de films sont sortis dans les années quatre - vingt(l’année de sortie est indiquée entre parenthèses à la fin du titre de chaque film).

        db.Films.countDocuments({ "titre": { $regex: "(198)" } })

1. Donner le nombre de de films d’horreur.

        db.Films.countDocuments({ "genres": { $regex: "Horror" } })

1. Modifier la collection utilisateurs en remplaçant pour chaque utilisateur le champ timestamp par un nouveau champ date, de type Date.

        db.Utilisateurs.updateMany(
            {},
            [{
                $set: {
                    films: {
                        $map: {
                            input: "$films",
                            in: {

                                idfilm: "$$this.idfilm",
                                note: "$$this.note",
                                date: {
                                    "$toDate": {
                                        "$multiply": [
                                            {
                                                $toLong: "$$this.timestamp"
                                            },
                                            1000
                                        ]
                                    }
                                },
                            }
                        }
                    }
                }
            }],
            { multi: true }
        )


1. Donner le nombre d’utilisateurs ayant noté plus de 90 films.

        db.Utilisateurs.aggregate(
            [
                {
                    $project: {
                        _id: 1, films: 1,
                        size_of_films: { $size: "$films" }
                    }
                },
                { $match: { "size_of_films": { $gt: 90 } } },
                {
                    $count: "number_of_users"
                }
            ])

1. Donner le nombre de notes soumises après le 2 janvier 2002.

        db.Utilisateurs.countDocuments({
            "films.date": {
                $gt: ISODate("2002-01-02T00:00:00.000Z")
            }
        })

1. Afficher trois derniers films notés par Jayson Brad.

        db.Utilisateurs.aggregate(
            { $match: { nom: "Jayson Brad" } },
            { $unwind: '$films' },
            { $sort: { 'films.date': -1 } },
            { $limit: 3 },
        )

1. L’utilisateur Venetta Graham vient de regarder le film Four Rooms qui a pour id 18 et lui donne la note de 3. Mettre à jour la base de données pour comptabiliser cette note.

        db.Utilisateurs.updateOne(
            { nom: "Venetta Graham" },
            {
                $push: {
                    films:
                    {
                        "idfilm": 18, "note": 3,
                        "date": new Date()
                    }
                },
            })


1. Montrer combien de films ont été produits durant chaque année des années 80; ordonner les résultats de l’année la plus à la moins fructueuse.

        db.Films.aggregate(
            [
                {
                    $project:
                    {
                        _id: 1,
                        length: { $strLenCP: "$titre" },
                        year: { $substrCP: ["$titre", { $subtract: [{ $strLenCP: "$titre" }, 6] }, 6] },
                    }
                },
                { $match: { year: { $regex: "(198)" } } },
                { $group: { _id: "$year", nombre_de_films: { $sum: 1 } } },
                { $sort: { nombre_de_films: -1 } }

            ]
        )

1. Déterminer la note moyenne du film « Terminator 2: Judgment Day », qui a comme id 589.

        db.Utilisateurs.aggregate([
            {
                $match: {
                    "films.idfilm": db.Films.findOne({ "titre": { $regex: "Terminator 2: Judgment Day" } }, { _id: 1 })["_id"]
                }
            },
            {
                $project: {
                    films: {
                        $filter: {
                            input: "$films",
                            as: 'film',
                            cond: { $eq: ["$$film.idfilm", db.Films.findOne({ "titre": { $regex: "Terminator 2: Judgment Day" } }, { _id: 1 })["_id"]] }, //
                        }
                    }
                }
            },
            { $unwind: "$films" },
            {
                $project:
                {
                    // _id: 1,
                    note: "$films.note",
                }
            },
            {
                $group:
                {
                    _id: 1,
                    avgNote: { $avg: "$note" }
                }
            },
        ])

1. En une seule requête, retourner pour chaque utilisateur son id, son nom, les notes maximale, minimale et moyenne qu’il a données, et ordonner le résultat par note moyenne croissante.

        db.Utilisateurs.aggregate([
            {
                $project: {
                    nom: 1,
                    note_min: { $min: "$films.note" },
                    note_max: { $max: "$films.note" },
                    note_moy: { $avg: "$films.note" },
                }
            },
            { $sort: { note_moy: 1 } },
        ])
        
