---
layout: default
---

# Rapport Neo4J / Gephi
<br>

## Movies 

### Aperçu de la base de données Movies 

|Schéma de Movies| Extrait de Movies|
|-|-|
|![alt](https://user-images.githubusercontent.com/104632559/194583663-6d8488d3-dd0c-4ac7-ae38-b60a96b9e29d.svg) | ![alt](https://user-images.githubusercontent.com/104632559/194583669-79b1a628-4cea-4211-b340-f3404793d2c7.svg)

<br>

### Requête simple
Cette requête renvoie les personnes ayant eu différents postes dans leurs carrières. 
```sql
MATCH (p:Person)-[:ACTED_IN]->(m:Movie)
WHERE exists((p)-[:DIRECTED]->())
OR exists( (p)-[:PRODUCED]->() )
RETURN distinct *
```
![alt](https://user-images.githubusercontent.com/104632559/194765325-258db45e-e1c6-4de9-9971-6e456fc252e8.svg)

<br>

### La requête Kevin Beacon :
```sql
MATCH d = shortestPath( (p:Person {name: 'Kevin Bacon'})-[*]-(hollywood WHERE hollywood.name <> 'Kevin Bacon'))
RETURN hollywood.name  as Actor , length(d) as DegreeOfSeparation 
ORDER BY DegreeOfSeparation
```
Cette requête renvoie un tableau avec le nom des acteurs et leur degré de séparation avec Kevin Bacon
Voici un extrait de la table de résultat obtenu classé par degrés de séparation avec Kevin Bacon


||Actor            |DegreeOfSeparation|
|------|-----------------|------------------|
|0     |Tom Cruise       |2                 |
|1     |Jack Nicholson   |2                 |
|2     |Demi Moore       |2                 |
|3     |Kiefer Sutherland|2                 |
|4     |Noah Wyle        |2                 |

<br>

On peut aussi obtenir un graphe représentant les éléments ayant une relation de distance 3 avec Kevin Bacon avec la requête suivante : 
```sql
MATCH (bacon:Person {name:"Kevin Bacon"})-[r:ACTED_IN*1..3]-(hollywood)     
RETURN DISTINCT hollywood, bacon
```
qui donne le résultat suivant dans lequel on voit clairement que Kevin Bacon est au centre du graphe : 
![alt](https://user-images.githubusercontent.com/104632559/201224208-77a464df-6f7b-497a-b4df-84205c9483be.svg)

<br>

---
<br>

## Air routes 

On peut aussi représenter le schéma de la base de données pour la base de données Airport Routes avec la commande :
```sql
CALL db.schema.visualization()
```

Ce qui donne ce résultat  : 

![alt](https://user-images.githubusercontent.com/104632559/201704223-0184a075-6758-41d1-848d-7b7babf7132c.svg)

### Requêtes basiques
- Requête pour obtenir le nombre d'aéroports par continents et par pays : 
```sql
MATCH (:Airport)-[:ON_CONTINENT]->(c:Continent)
RETURN c.name AS continentName, count(*) AS numAirports ORDER BY numAirports DESC
```
|continentName|numAirports|
|-------------|-----------|
|NA           |989        |
|AS           |971        |
|EU           |605        |
|AF           |321        |
|SA           |313        |
|OC           |304        |

```sql
MATCH (:Airport)-[:IN_COUNTRY]->(c:Country)
RETURN c.code AS CountryName, count(*) AS numAirports ORDER BY numAirports DESC
```
|CountryName|numAirports|
|-----------|-----------|
|US         |586        |
|CN         |217        |
|CA         |205        |
|AU         |131        |
|RU         |129        |

- Requêtes pour obtenir le nombre de route par aéroport : 
```sql
MATCH (a:Airport)<-[HAS_ROUTE]-(a1:Airport)
RETURN a.iata AS Identifiant, a.descr AS Name, COUNT(a1) AS NbOfRoutes
ORDER BY NbOfRoutes DESC
```

On obtient alors le tableau suivant 
|Identifiant|Name                          |NbOfRoutes|
|-----------|------------------------------|----------|
|FRA        |Frankfurt am Main             |303       |
|CDG        |Paris Charles de Gaulle       |291       |
|AMS        |Amsterdam Airport Schiphol    |280       |
|IST        |Istanbul International Airport|268       |
|MUC        |Munich International Airport  |265       |

- Requête pour obtenir les aéroports reliés à l'aéroport d'Atlanta classés par ordre décroissant : 
```sql
MATCH (a:Airport {iata : 'ATL'})-[d:HAS_ROUTE]->(a1:Airport)
RETURN a.iata AS Identifiant, a.descr AS Name, d.distance AS Distance, a1.descr 
ORDER BY Distance DESC
```
On obtient alors le csv suivant : 
|Identifiant|Name                          |Distance|a1.descr                    |
|-----------|------------------------------|--------|----------------------------|
|ATL        |Hartsfield - Jackson Atlanta International Airport|8434    |Johannesburg, OR Tambo International Airport|
|ATL        |Hartsfield - Jackson Atlanta International Airport|7640    |Shanghai - Pudong International Airport|
|ATL        |Hartsfield - Jackson Atlanta International Airport|7581    |Dubai International Airport |
|ATL        |Hartsfield - Jackson Atlanta International Airport|7442    |Doha, Hamad International Airport|
|ATL        |Hartsfield - Jackson Atlanta International Airport|7133    |Seoul, Incheon International Airport|

### Requête de plus court chemin 
Pour obtenir le plus court chemin entre deux noeuds on peut utiliser la commande suivante : 

```sql
MATCH (a:Airport { city:"New York" }),(a1:Airport { city: "Paris" }), 
p = shortestPath((a)-[* .. ]->(a1))
RETURN a.descr as Airport_NY, a1.descr as Airport_Paris, length(p) AS NbOfFlights, [x in nodes(p) | x.iata] AS Path ORDER BY Length(p)
```
Cette formule renvoie le csv suivant : 
|Airport_NY|Airport_Paris                 |NbOfFlights|Path                        |
|----------|------------------------------|-----------|----------------------------|
|New York John F. Kennedy International Airport|Paris Charles de Gaulle       |1          |[JFK,CDG]                   |
|New York John F. Kennedy International Airport|Paris, Orly Airport           |1          |[JFK,ORY]                   |
|New York La Guardia|Paris Charles de Gaulle       |2          |[LGA,IAH,CDG]               |
|New York La Guardia|Paris, Orly Airport           |2          |[LGA,YUL,ORY]               |

Néanmoins, cette requête ne prend pas en compte le poids de chaque route. Pour cela, il nous faut utiliser la projection weighted graph routes et  l’algorithme de Dijkstra.
Pour créer la projection routes-weighted on utilise la commande suivante :
```sql
CALL gds.graph.project(
    'routes-weighted',
    'Airport',
    'HAS_ROUTE',
    {
        relationshipProperties: 'distance'
    }
) 
YIELD
    graphName, nodeProjection, nodeCount, relationshipProjection, relationshipCount
```

Puis on utilise la commande suivante pour retourner le plus court chemin entre Paris et Auckland : 
```sql

```

![alt](https://user-images.githubusercontent.com/104632559/201995263-1e8c31e0-148c-493e-9a4d-5e94063fdaad.svg)

On obtient aussi le csv suivant qui nous donne les informations sur les chemins possibles et le coût total du trajet : 

|sourceNodeName                |targetNodeName|totalCost                   |nodeNames        |costs                     |
|------------------------------|--------------|----------------------------|-----------------|--------------------------|
|CDG                           |AKL           |11516.0                     |[CDG,NRT,AKL]    |[0.0,6029.0,11516.0]      |
|ORY                           |AKL           |11541.0                     |[ORY,CPH,HND,AKL]|[0.0,644.0,6051.0,11541.0]|

On peut comparer les résultas de l'algorithme de Dijkstra avec l'algorithme du ShortestPath de Neo4j utilisé plus haut : 

- Graphe : 
![alt](https://user-images.githubusercontent.com/104632559/202003802-a4f9fbf8-b356-48e5-b128-6e710312b751.svg)

- csv : 

|sourceNodeName                |targetNodeName|totalCost                   |nodeNames        |costs                     |
|------------------------------|--------------|----------------------------|-----------------|--------------------------|
|JFK                           |CDG           |3622.0                      |[JFK,CDG]        |[0.0,3622.0]              |
|JFK                           |ORY           |3622.0                      |[JFK,ORY]        |[0.0,3622.0]              |
|LGA                           |CDG           |3620.0                      |[LGA,BOS,CDG]    |[0.0,184.0,3620.0]        |
|LGA                           |ORY           |3661.0                      |[LGA,BOS,DUB,ORY]|[0.0,184.0,3166.0,3661.0] |



### Visualisation des aéroports avec le PageRank
En utilisant la projection graph routes on obtient le CSV suivant qui nous donne les communautés générés par l'algorithme du PageRank :

```sql
CALL gds.pageRank.stream('routes')
YIELD nodeId, score
WITH gds.util.asNode(nodeId) AS n, score AS pageRank
RETURN n.iata AS iata, n.descr AS description, pageRank
ORDER BY pageRank DESC, iata ASC
```

|iata|description                                                                  |pageRank           |
|----|-----------------------------------------------------------------------------|-------------------|
|DFW |Dallas/Fort Worth International Airport                                      |11.97978260670334  |
|ORD |Chicago O'Hare International Airport                                         |11.162988178920267 |
|DEN |Denver International Airport                                                 |10.997299338126385 |
|ATL |Hartsfield - Jackson Atlanta International Airport                           |10.389948350302957 |
|IST |Istanbul International Airport                                               |8.425801217705779  |
|CDG |Paris Charles de Gaulle                                                      |8.401469085296542  |

Néanmoins, on peut aussi visualiser le graphe pour mettre l'accent sur les aéroports ayant le plus d'importance dnas le reseau : 

![alt](https://user-images.githubusercontent.com/104632559/201896433-e83b93cd-d694-4968-b6eb-fbe3a40b1616.png)


### Visualisation des communautés de Louvain
En utilisant la projection graph routes on obtient le CSV suivant qui nous donne les communautés générés par l'algorithme de Louvain : 
```sql
CALL gds.louvain.stream('routes')
YIELD nodeId, communityId
WITH gds.util.asNode(nodeId) AS n, communityId
RETURN
	communityId,
    SIZE(COLLECT(n)) AS numberOfAirports,
	COLLECT(DISTINCT n.city) AS cities
ORDER BY numberOfAirports DESC, communityId
```

|communityId|numberOfAirports|cities                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
|-----------|----------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|3321       |699             |[Atlanta,Anchorage,Austin,Nashville,Boston,Baltimore,Washington D.C.,Dallas,Fort Lauderdale,Houston,New York,Los Angeles,Orlando,Miami,Minneapolis,Chicago,West Palm Beach,Phoenix,Raleigh,Seattle,San Francisco,San Jose,Tampa,San Diego,Long Beach,Santa Ana,Salt Lake City,Las Vegas,Denver,White Plains,San Antonio,New Orleans,Newark,Cedar Rapids,Honolulu,El Paso,San Juan,Cleveland,Oakland,Tucson,Santa Fe,Philadelphia,Detroit,Toronto,Vancouver,Ottawa,Fort Myers,Montreal,Edmonton,Calgary,St. John's,Mexico City,Kingston,Tallahassee,Pittsburgh,Portland,Oaklahoma City,Ontario,Rochester,Halifax,Winnipeg,Charlotte,Cancun,Palm Springs,Memphis,Cincinnati,Indianapolis,Kansas City,St Louis,Albuquerque,Milwaukee,Harrison,Salina,Omaha,Tulsa,Puerto Vallarta,Kahului,Nassau,Freeport,George Town,Key West,Bridgetown,St. George,Charlotte Amalie,Hamilton,Scarborough,Port of Spain,Montego Bay,Little Rock,Kralendijk,Oranjestad,Norfolk,Jacksonville,Providence,Punta Cana,Harrisburg,Sacramento,Roatan Island,Tegucigalpa,Colorado Springs,Huntsville,Birmingham,Quebec City,Rapid City,Louisville,Buffalo,Shreveport,Boise,Lihue,Lubbock,Panama City Beach,Harlingen,Reno,Columbus,Idaho Falls,Albany,Wichita,Midland,Saskatoon,Hartford,Billings,Sint Martin,Springfield,Richmond,Texarkana,Peoria,Hilo,Lexington,Guatemala City,Islip,Niagara Falls,Newburgh,South Bimini,Havana,Corpus Christi,Abilene,Waco,College Station,Bloomington/Normal,Beaumont/Port Arthur,Des Moines,Myrtle Beach,Alexandria,Cozumel,Aguascalientes,Monterrey,Amarillo,Silao,Brownsville,Baton Rouge,Belize City,Columbia,Chattanooga,Charleston,Champaign/Urbana,Dayton,Chihuahua,Durango,Evansville,Fargo,Fresno,Sioux Falls,Fort Smith,Fort Wayne,Garden City,Guadalajara,Longview,Grand Junction,Gulfport,Grand Island,Fort Hood/Killeen,Grand Rapids,Greensboro,Greenville,Jackson,Joplin,Lawton,Lake Charles,Lafayette,Liberia,Laredo,Mc Allen,Montgomery,Manhattan,Moline,Morelia,Monroe,Mobile,Madison,Mazatlán,Puebla,Providenciales Island,Pensacola,Querétaro,Roswell,Santa Clara,Savannah,San José del Cabo,San Angelo,San Luis Potosí,Wichita Falls,Torreón,Tyler,Knoxville,Valparaiso,Fayetteville/Springdale/,Zacatecas,Vieux Fort,Akron,Burlington,Manchester,Syracuse,Regina,Florence,Asheville,Eagle,Hayden,Sarasota/Bradenton,Topeka,Lansing,Roanoke,Marquette,Green Bay,Augusta,Bangor,Fayetteville,Hilton Head Island,Wilmington,Puerto Plata,Holguin,Varadero,Aguadilla,Naples,Gainesville,La Romana,Santo Domingo,Santiago,La Mesa,Mérida,Managua,Port-au-Prince,Cayman Brac,Georgetown,Marsh Harbour,North Eleuthera,Cartagena,Fort-de-France,Pointe-à-Pitre Le Raizet,Saint George's,Christiansted,Basseterre,Burbank,Samana,Ponce,Atlantic City,Allentown,Appleton,Wilkes-Barre/Scranton,Kalamazoo,Brunswick,Charlottesville,Daytona Beach,Dothan,New Bern,Flint,Columbus/W Point/Starkville,Lewisburg,Saginaw,Macon,Meridian,Melbourne,Muscle Shoals,Newport News,Hattiesburg/Laurel,South Bend,Bristol/Johnson/Kingsport,Trenton,Tupelo,Valdosta,Lanai City,Kailua/Kona,Pago Pago,Lahaina,Kaunakakai,London,Waterloo,Sioux City,Kitchener,Cayo Coco,Victoria,Arcata/Eureka,Bakersfield,Crescent City,Chico,Eugene,Klamath Falls,Medford,Modesto,Monterey,North Bend,Pasco,Redding,Redmond,Santa Barbara,San Luis Obispo,Kelowna,Aspen,Bellingham,Carlsbad,Spokane,Kingman,Merced,Mammoth Lakes,Yuma,Prescott,Santa Maria,Santa Rosa,Visalia,Hermosillo,Loreto,Uruapan,Ixtapa,Manzanillo,Provo,Butte,Bozeman,Cedar City,Moab,Cody,Casper,Elko,Gillette,Kalispell,Great Falls,Helena,Lewiston,Missoula,Pocatello,Rock Springs,St George,Twin Falls,Vernal,Branson,Fort McMurray,Alliance,Alamosa,Scottsbluff,Bismarck,Cortez,Cheyenne,Dodge City,Dickinson,Kearney,Farmington,Gunnison,Williston,Laramie,North Platte,Liberal,Lincoln,Mc Cook,Minot,Montrose,Page,Pierre,Pueblo,Riverton,Sheridan,Watertown,Hancock,Mosinee,Dubuque,Decatur,Duluth,Eau Claire,Elmira/Corning,La Crosse,Muskegon,Paducah,St Cloud,Toledo,Traverse City,State College,Toluca,Latrobe,Worcester,Plattsburgh,Treasure Cay,Governor's Harbour,San Salvador,Moncton,Altoona,Binghamton,Beckley,Hagerstown,Johnstown,Lancaster,Morgantown,Staunton/Waynesboro/Harrisonburg,Ithaca,Grand Forks,Chicago/Rockford,Aberdeen,Alpena,Bemidji,Brainerd,Hibbing,Iron Mountain / Kingsford,International Falls,Rhinelander,Nantucket,Bar Harbor,Hyannis,Lebanon,Martha's Vineyard,Presque Isle,Provincetown,Rockland,Rutland,Saranac Lake,Cold Bay,Cordova,Adak Island,Dillingham,Kenai,Homer,Iliamna,King Salmon,Sand Point,St Mary's,St Paul Island,Valdez,Athens,Hobbs,Acapulco,Huatulco,Ciudad del Carmen,Saltillo,Oaxaca,Tampico,Villahermosa,Veracruz,Flagstaff,Show Low,Silver City,Sault Ste Marie,Deer Lake,Fredericton,Windsor,Thunder Bay,Sydney,Sudbury,Saint John,Timmins,North Bay,Charlottetown,Sarnia,Camaguey,Erie,New Haven,Williamsport,Salisbury,Escanaba,Pellston,Bradford,Dubois,Franklin,Jamestown,Parkersburg,Imperial,Trail,Bella Coola,Campbell River,Nanaimo,Castlegar,Dawson Creek,Kamloops,Prince Rupert,Powell River,Comox,Quesnel,Williams Lake,Cranbrook,Fort St.John,Prince George,Terrace,Whitehorse,Smithers,Penticton,Sandspit,Port Hardy,Masset,Walla Walla,Wenatchee,St Petersburg-Clearwater,Pullman/Moscow,Yakima,Marigot,Gustavia,Culebra Island,Mayaguez,Vieques Island,Charlestown,The Valley,Road Town,Spanish Town,Stockton,Punta Gorda,Baie-Comeau,Bagotville,High Level,Rainbow Lake,Grande Prairie,Wabush,Abbotsford,Mont-Joli,Sept-Îles,Bathurst,Saint-Pierre,Tijuana,Cayo Largo del Sur,Fort Nelson,Culiacán,Chetumal,Ciudad Obregón,Campeche,Ciudad Juárez,Ciudad Victoria,Tepic,Colima,Xalapa,Lázaro Cárdenas,Los Mochis,La Paz,Matamoros,Mexicali,Minatitlán,Nuevo Laredo,Poza Rica,Piedras Negras,Palenque,Puerto Escondido,Reynosa,Tuxtla Gutiérrez,Tapachula,Brandon,Lloydminster,Red Deer,Lethbridge,Medicine Hat,Chadron,Levelock,Anahim Lake,St. Anthony,Charlo,Fort Hope,Gaspé,Îles-de-la-Madeleine,Havre St-Pierre,Montréal,Bella Bella,Stephenville,Schefferville,Moosonee,Natashquan,Port-Menier,Gander,Bonaventure,Webequie,Goose Bay,Kapuskasing,Churchill Falls,Belleville,Cape Girardeau,Clarksburg,El Dorado,New Bedford,Grand Canyon,Glendive,Glasgow,Huron,Hot Springs,Huntington,Havre,Kirksville,Jonesboro,Los Alamos,Lynchburg,Manistee,Morristown,Massena,Marion,Ogdensburg,Wolf Point,Owensboro,Pendleton,Portsmouth,Sidney,Fort Leonard Wood,Teterboro,Thief River Falls,Quincy,Worland,Youngstown/Warren,Cockburn Town,South Caicos Island,La Isabela,San Benito,La Ceiba,Puerto Lempira,Isla Colón,David,La Fortuna/San Carlos,Roxana,Puntarenas,Golfito,Nicoya,Puerto Jimenez,Palmar Sur,Quepos,Santa Cruz,Cap Haitien,Guantánamo,Little Cayman,Spring Point,Arthur's Town,Colonel Hill,Rock Sound,Matthew Town,Deadman's Cay,Stella Maris,Mayaguana,Port Nelson,False Pass,Mountain Village,Pilot Point,King Cove,Yakutat,Kotlik,South Naknek,Sheldon Point,Castries,Kingstown,Igiugig,Lansdowne House,St Augustine,Fort Albany,Ogden,La Romaine,Ogoki Post,Chevery,Lourdes-De-Blanc-Sablon,Salt Cay,Albrook,Nelson Lagoon,Kegaska,Concord,Hailey,Everett,Ceiba]|
|2294       |518             |[London,Paris,Frankfurt,Helsinki,Dublin,Rome,Amsterdam,Prague,Barcelona,Madrid,Vienna,Zurich,Geneva,Brussels,Munich,Manchester,Cologne,Gothenburg,Venice,Shannon,Oslo,Stockholm,Nottingham,Edinburgh,Glasgow,Liverpool,Nice,Milan,Athens,Zagreb,Budapest,Alicante,Bilbao,Ibiza,Menorca,Tenerife,Larnaca,Warsaw,Luqa,Sofia,Belgrade,Tel Aviv,Hamburg,Stuttgart,Genoa,Naples,Pisa,Turin,Bologna,Verona,Nantes,Copenhagen,Luxembourg,Dusseldorf,Lisbon,Gibraltar,Tunis,Reykjavik,Gran Canaria,Southampton,Palma De Mallorca,Riga,Malaga,Funchal,Leeds,Aberdeen,Antalya,Saint Helier,Zakynthos,Rhodes,Bristol,Newcastle,Saint Peter Port,Eindhoven,Sevilla,Basle,Dubrovnik,Stavanger,Bergen,Tallinn,Algiers,Cork,Wroclaw,Split,Belfast,Hannover,Lyon,Marseille,Bucharest,Rotterdam,Casablanca,Tangier,Faro,Mykonos Island,Santorini Island,Kiev,Rijeka,Toulouse/Blagnac,Porto,Culleredo,Innsbruck,Birmingham,Inverness,Salzburg,Kos Island,Trondheim,Billund,Bern,Castletown,Granada,Firenze,Dresden,Deauville,Brive,Brest/Guipavas,Antwerp,Bremen,Clermont-Ferrand/Auvergne,Prishtina,Hassi Messaoud,Erfurt,Newquay,Charleston,Aalborg,Ålesund,Torp,Kraków,Kaunas,Fuerteventura Island,Lanzarote Island,Tenerife Island,Agadir,Marrakech,Espargos,Rabil,Hurghada,Sharm el-Sheikh,Tirana,Paphos,Almería,San Javier,Santiago de Compostela,Valencia,Bordeaux/Mérignac,Bastia/Poretta,Ajaccio/Napoléon Bonaparte,Montpellier/Méditerranée,Strasbourg,Heraklion,Kefallinia Island,Kalamata,Kerkyra Island,Preveza/Lefkada,Souda,Thessaloniki,Bari,Catania,Palermo,Olbia,Ponta Delgada,Dalaman,Bodrum,Tivat,Enfidha,Dortmund,Nuremberg,Leipzig,Cardiff,Southend,Grimsby,Durham,Norwich,Exeter,Kjevik,Gdańsk,Växjö,Linköping,Sta Cruz de la Palma, La Palma Island,Nador,São Pedro,Girona,Chios Island,Kithira Island,Samos Island,Lamezia Terme,Ljubljana,Konya,Kayseri,Killarney,Katowice,Lublin,Vilnius,Burgas,Reus,Béziers/Vias,Varna,Nîmes/Garons,Waterford,Rygge,Rzeszów,Pula,Zadar,Beauvais/Tillé,Trapani,Bergamo,Roma,Djerba,Heringsdorf,Münster,Friedrichshafen,Westerland,Poznań,Jerez de la Forntera,Patras,Graz,Linz,Chişinău,Podgorica,Skopje,Bratislava,Annabah,Constantine,Oran,Monastir,Rabat,Ranón,Vigo,Pau/Pyrénées (Uzein),Biarritz/Anglet/Bayonne,Calvi/Sainte-Catherine,Rennes/Saint-Jacques,Ostrava,Kharkiv,Melilla,Badajoz,Logroño,Pamplona,Hondarribia,Santander,Tarbes/Lourdes/Pyrénées,Alghero,Cagliari,Cluj-Napoca,Timişoara,Marsa Alam,Nea Anchialos,Klagenfurt am Wörthersee,Sarajevo,Iaşi,Sibiu,Altenrhein,Košice,Lviv,Lille/Lesquin,Brindisi,Lugano,Marina  Di Campo,Banja Luka,Limoges/Bellegarde,Metz / Nancy,Figari Sud-Corse,Caen/Carpiquet,Hahn,Memmingen,Bournemouth,Blackpool,Donegal,Bydgoszcz,Goleniow,La Rochelle/Île de Ré,Rodez/Marcillac,Carcassonne/Salvaza,Perpignan/Rivesaltes,Tours/Val de Loire (Loire Valley),Comiso,Tartu,Ivalo,Joensuu / Liperi,Jyväskylän Maalaiskunta,Kemi / Tornio,Kajaani,Kokkola / Kruunupyy,Kuusamo,Kuopio / Siilinjärvi,Mariehamn,Oulu / Oulunsalo,Pori,Rovaniemi,Savonlinna,Tampere / Pirkkala,Turku,Vaasa,Norrköping,Paderborn,Weeze,Derry,Campbeltown,Lerwick,Wick,Port Ellen,Balivanich,Stornoway,Eoligarry,Balemartine,Stockholm / Nyköping,Rostock,Fes,Oujda,Burgos,León,Salamanca,Valladolid,Bergerac/Roumanière,Poitiers/Biard,Karpathos Island,Kavala,Mytilene,Lampedusa,Pantelleria,Reggio Calabria,Trieste,Ancona,Eskişehir,Bolzano,Aarhus,Førde,Brønnøy,Målselv,Leirin,Florø,Karmøy,Kvernberget,Årø,Ørland,Ørsta,Røros,Sandane,Sogndal,Leirvik,Palanga,Baden-Baden,Dundee,Malmö,Skellefteå,Stockholm / Västerås,Plovdiv,Osijek,Zaragoza,Dinard/Pleurtuit/Saint-Malo,Toulon/Hyères/Le Palyvestre,Pescara,Parma,Perugia,Brno,Sundsvall/ Härnösand,Borlange,Ronneby,Jönköping,Mora,Kristianstad,Kalmar,Halmstad,Sveg,Gällivare,Kramfors / Sollefteå,Lycksele,Örnsköldsvik,Kiruna,Umeå,Östersund,Hagfors,Karlstad,Luleå,Visby,Ängelholm,Kittila,Béjaïa,Sétif,Batna,Tlemcen,Biskra,Tozeur,Essaouira,Ouarzazate,Agen/La Garenne,Périgueux/Bassillac,Castres/Mazamet,Le Puy/Loudes,Aurillac,Lorient/Lann/Bihoué,Lannion,Quimper/Pluguffan,Gafsa,Gabès,Liège,Ostend,Saarbrücken,Lübeck,Zweibrücken,Kassel,Doncaster,Maastricht,Groningen,Lleida,Dole/Tavaux,Alexandroupolis,Ikaria Island,Ioannina,Kastoria,Kastelorizo Island,Kalymnos Island,Kozani,Leros Island,Limnos Island,Milos Island,Naxos Island,Paros Island,Astypalaia Island,Skiathos,Syros Island,Crete Island,Skiros Island,Vila do Porto,Vila Baleira,Djanet,Mecheria,Tébessi,Hassi R'Mel,Tiaret,Chlef,Béchar,Mascara,El Bayadh,Adrar,Ghardaïa,In Salah,Touggourt,Guemar,Ouargla,Aménas,Mannheim,Augsburg,Kuressaare,Lappeenranta,Staverton,St. Mary's,Saint Anne,Hawarden,Angelsey,Esbjerg,Karup,Rønne,Sønderborg,Vagar,Geiteryggen,Trollhättan,Arvidsjaur,Örebro,Torsby,Pajala,Alajero, La Gomera Island,El Hierro Island,Dakhla,El Aaiún,Tetuan,Preguiça,Brač Island,Saint-Étienne/Bouthéon,Avignon/Caumont,Le Havre/Octeville,Châlons/Vatry,Dijon/Longvic,Kasos Island,Debrecen,Cuneo,Eilat,Mostar,Tuzla,Arad,Baia Mare,Constanţa,Craiova,Oradea,Satu Mare,Târgu Mureş,Zonguldak,Goulimime,Aqaba,Kutaisi,Al Hoceima,Tamanrasset,Land's End,Haifa,Nis,Tan Tan,Zhukovsky,Szcytno,Chambéry/Aix-les-Bains,Grenoble,Valladolises,Berlin]                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
|2784       |383             |[Dubai,New Delhi,Mumbai,Doha,Calicut,Hyderabad,Chennai,Kolkata,Bengaluru,Cape Town,Johannesburg,Durban,Nairobi,Mombasa,Cairo,Addis Ababa,Kuwait,Istanbul,Manama,Abu Dhabi,Colombo,Kathmandu,Jeddah,Muscat,Lagos,Harare,Luxor,Riyadh,Islamabad,Amman,Karachi,Lahore,Jaipur,Accra,Kampala,Abuja,Beirut,Freetown,Luanda,Tripoli,Dhaka,Sylhet,Tehran,Port Louis,Mahe Island,İzmir,Malé,Kigali,Arusha,Ad Dammam,Arbil,Dakar,Agra,Khajuraho,Varanasi,Mangalore,Ahmedabad,Jodhpur,Pune,Sharjah,Aden,Coimbatore,Cochin,Trivandrum,Tiruchirappally,Sana'a,Diu,Porbandar,Windhoek,Ankara,Lusaka,Hargeisa,Berbera,Djibouti City,Alexandria,Port Sudan,Juba,Khartoum,Dar es Salaam,Gheshm,Kabul,Douala,Cotonou,Ouagadougou,Abidjan,Niamey,Sfax,Lomé,Brazzaville,Pointe Noire,Bangui,Yaoundé,St Denis,Antananarivo,Libreville,N'Djamena,Kinshasa,Senou,Monrovia,Nouakchott,Conakry,Praia,Visakhapatnam,Kandahar,Abha,Buraidah,Ha'il,Medina,Tabuk,Ta’if,Yenbo,Ahwaz,Bushehr,Kish Island,Bandar Lengeh,Isfahan,Bandar Abbas,Mashhad,Lar,Lamerd,Shiraz,Tabriz,Chabahar,Zahedan,Salalah,Multan,Peshawar,Sialkot,Baghdad,Basrah,Najaf,Sulaymaniyah,Riyan,Vasco da Gama,Chittagong,Lucknow,Sir Bani Yas Island,Jebel Ali,Amritsar,Malabo,Herat,Aurangabad,Vadodara,Bhopal,Indore,Jabalpur,Naqpur,Raipur,Magdalla,Udaipur,Siliguri,Bhubaneswar,Gorakhpur,Guwahati,Dibrugarh,Patna,Ranchi,Allahabad,Bhuntar,Chandigarh,Dehradun,Kangra,Jammu,Kanpur,Ludhiana,Leh,Srinagar,Banjul,Bujumbura,Donetsk,Dzaoudzi,Bhuj,Bhavnagar,Hubli,Jamnagar,Rajkot,Gwalior,Bloemfontain,East London,Ellisras,George,Hoedspruit,Kimberley,Mpumalanga,Margate,Port Elizabeth,Plettenberg Bay,Phalaborwa,Pietermaritzburg,Potgietersrus,Richards Bay,Upington,Mthatha,Francistown,Kasane,Maun,Gaborone,Manzini,Livingstone,Ndola,Beira,Inhambabe,Maputo,Nampula,Pemba / Porto Amelia,Tete,Vilanculo,Bulawayo,Victoria Falls,Blantyre,Lilongwe,Maseru,Walvis Bay,Lubumbashi,Zanzibar,São Tomé,Sirt,Tobruk,Benghazi,Al Bayda,Misratah,Bobo Dioulasso,Tamale,Kumasi,Sunyani,Sekondi-Takoradi,Uyo,Asaba,Benin,Calabar,Enegu,Ibadan,Ilorin,Owerri,Jos,Kaduna,Kano,Port Harcourt,Sokoto,Yola,Bata,Port Mathurin,Maroua,N'Gaoundéré,Garoua,Chipata,Kasama,Mansa,Mfuwe,Solwesi,Moroni,St Pierre,Sainte Marie,Toamasina,Morondava,Antsiranana,Antalaha,Mahajanga,Nosy Be,Maroantsetra,Sambava,Tôlanaro,Tulear,Mbanza Congo,Cabinda,Catumbela,Ngiva,Huambo,Kuito,Malanje,Menongue,Namibe,Saurimo,Soyo,Lubango,Luena,Port Gentil,Chimoio,Lichinga,Quelimane,Praslin Island,Mbandaka,Boende,Kisangani,Kindu,Kananga,Tshikapa,Mbuji Mayi,Bissau,Ziguinchor,Nouadhibou,Zouérate,Vila do Maio,São Filipe,Arba Minch,Axum,Bahir Dar,Dire Dawa,Gambela,Gondar,Jijiga,Jimma,Makale,Asosa,Bosaso,Mogadishu,Galkayo,Assiut,Sohag,Aswan,Asmara,Eldoret,Kisumu,Lodwar,Malindi,Wajir,Ghat,Kufra,Ghadames,Kamembe,El Fasher,Geneina,Nyala,Mbeya,Mtwara,Mwanza,Arua,Kasese,Pakuba,Nicosia,Adana,Gaziantep,Kastamonu,Amasya,Sivas,Malatya,Denizli,Nevşehir,Çanakkale,Kosekoy,Çorlu,Elazığ,Diyarbakir,Erzincan,Erzurum,Kars,Trabzon,Van,Batman,Muş,Kahramanmaraş,Agri,Adıyaman,Mardin,Şanlıurfa,Hatay,Isparta,Edremit,Ubari,Gombe,Warri,Al Ahsa,Al-Fujairah,Quetta,Ta'izz,Paro,Semera,Bukavu,Gemena,Shire Indasilase,Damascus,Ras Al Khaimah,Kannur,Garowe,Wa]                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
|2838       |253             |[Tokyo,Singapore,Hong Kong,Beijing,Shanghai,Kuala Lumpur,Manila,Bangkok,Osaka,Seoul,Phnom Penh,Ho Chi Minh City,Hagta,Taipei,Fukuoka,Sapporo,Denpasar,Jakarta,Guangzhou,Phuket,Nanjing,Chengdu,Hanoi,Kaohsiung City,Tokoname,Xiamen,Hangzhou,Vladivostok,Shenzhen,Hiroshima,Dili,Tianjin,Changsha,Penang,Wuhan,Haikou,Kunming,Fuzhou,Ningbo,Qingdao,Chongqing,Guiyang,Nanning,Kagoshima,Oita,Kanazawa,Yonago,Matsuyama,Takamatsu,Niigata,Sendai,Jeju City,Busan,Naha,Saipan Island,Babelthuap Island,Yangon,Chiang Mai,Krabi,Na Thon (Ko Samui Island),Hat Yai,Da Nang,Bandar Seri Begawan,Lapu-Lapu City,Ulaanbaatar,Taipa,Kotamadya Balikpapan,Kota Kinabalu,Angeles City,Kota Baharu,Iloilo City,Siem Reap,Khabarovsk,Yuzhno-Sakhalinsk,Changchun,Dalian,Shenyang,Kuching,Miri,Kuantan,Ipoh,Langkawi,Kuala Terengganu,Kuala Lumpur Subang,Davao City,Vientiane,Surabaya,Bandung-Java Island,Mataram,Kalibo,Pekanbaru-Sumatra Island,Palembang-Sumatra Island,Sukarata(Solo)-Java Island,Semarang-Java Island,Ujung Pandang-Celebes Island,Yogyakarta-Java Island,Taipei City,Shirahama,Kobe,Obihiro,Hakodate,Kushiro,Ōzora,Nakashibetsu,Wakkanai,Ube,Monbetsu,Asahikawa / Hokkaidō,Miyazaki,Kitakyūshū,Saga,Kumamoto,Nagasaki,Amami,Toyama,Wajima,Okayama City,Izumo,Nankoku,Tottori,Tokushima,Masuda,Aomori,Yamagata,Akita,Misawa,Odate,Shonai,Hachijojima,Izu Oshima,Ishigaki,Miyako City,Taichung City,Tainan City,Taiyuan,Jinan,Xianyang,Nanchang,Zhengzhou,Yantai,Xuzhou,Sanya,Hailar,Daegu,Gwangju,Cheongju,Pyongyang,Lanzhou,Lhasa,Harbin,Jiamusi,Mudanjiang,Qiqihar,Yanji,Ji'an,Manado-Celebes Island,Redang,Shantou,Lijiang,Wuxi,Dongying,Dandong,Nantong,Daqing Shi,Huai'an,Qianjiang,Baishan,Chifeng,Changzhi,Ordos,Datong,Erenhot,,Hohhot,Baotou,Tongliao,Wuhai,Ulanhot,Xilinhot,Yuncheng,Beihai,Changde,Dayong,Guilin City,Zhuhai,Liuzhou,Zhanjiang,Luoyang,Xiangfan,Yichang,Yinchuan,Jining,Xining,Yan'an,Yulin,Zhongwei,Luxi,Anqing,Changzhou,Fuyang,Ganzhou,Jingdezhen,Jiujiang,Lianyungang,Huangyan,Linyi,Hefei,Quanzhou,Huangshan,Weifang,Wuyishan,Wenzhou,Yancheng,Yiwu,Zhoushan,Dazhou,Guangyuan,Luzhou,Meixian,Tangshan,Mohe,Baoshan,Lincang,Panzhihua,Foshan,Baise,Zhangye,Yichun,Kolaka/Pomala Tambea-Celebes Island,Jiagedaqi,Jinchang,Chizhou,Jixi,Heihe,Bijie,Sihanukville,Ürümqi,Nha Trang,Mandalay,Batam Island,Senai,Shijiazhuang,Mianyang,Zunyi,Yangzho,Medan,Weihai]                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
|2927       |187             |[Sydney,Melbourne,Perth,Auckland,Wellington,Brisbane,Christchurch,Canberra,Ayers Rock,Alice Springs,Gold Coast,Cairns,Maroochydore,Adelaide,Darwin,Nadi,Port Moresby,Papeete,Apia,Majuro Atoll,Banana,Tarawa,Yaren District,Nouméa,Avarua,Nausori,Nuku'alofa,Port Vila,Queenstown,Armidale,Broken Hill,Hamilton Island,Mackay,Ballina,Proserpine,Broome,Bathurst,Townsville,Gladstone,Griffith,Hervey Bay,Lord Howe Island,Lismore,Albury,Merimbula,Hobart,Mildura,Launceston,Moree,Moruya,Narrandera,Orange,Karratha,Parkes,Port Macquarie,Coffs Harbour,Dubbo,Burnt Pine,Tamworth,Wagga Wagga,Taree,Williamtown,Devonport,Currie,Mount Gambier,Kalgoorlie,Port Hedland,Burnie,Taupo,Dunedin,Gisborne,Hamilton,Kerikeri,Kaitaia,New Plymouth,Napier,Nelson,Palmerston North,Paraparaumu,Rotorua,Tauranga,Blenheim,Whakatane,Whangarei,Wanganui,Albany,Busselton,Derby,Esperance,Geraldton,Ravensthorpe,Newman,Paraburdoo,Kununurra,Exmouth,Christmas Island,Honiara,Luganville,Hokitika,Invercargill,Timaru,Westport,Barcaldine,Blackall,Charleville,Mount Isa,Rockhampton,Bundaberg,Cloncurry,Emerald,Longreach,Moranbah,Roma,Isla de Pascua (Easter Island),Winton,Arona,Atoifi,Anua,Choiseul Bay,Fera Island,Kirakira,Santa Cruz/Graciosa Bay/Luova,Munda,Gizo,Rennell Island,Sege,Santa Ana Island,Marau,Suavanao,Kagau Island,Ramata,Buka Island,Kundiawa,Daru,Goronka,Gurney,Popondetta,Hoskins,Kiunga,Kavieng,Londolovit,Madang,Mount Hagen,Mendi,Momote,Moro,Nadzab,Tari,Tabubil,Tokua,Vanimo,Wapenamanda,Wewak,Bulolo,Aitutaki,Atiu Island,Mangaia Island,Mauke Island,Mitiaro Island,Cicia,Vunisea,Lakeba Island,Labasa,Matei,Rotuma,Savusavu,Vanua Balavu,Funafuti,Wallis Island,Rimatara Island,Rurutu,Tubai,Raivavae,Hao,Lonorore,Norsup,Kwajalein,Grafton,Groote Eylandt,Kingscote,Cocos (Keeling) Islands,Nhulunbuy,Quilpie,Futuna Island,Toowoomba]                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |

On obtient alors en utilisant le module Bloom de neo4j qui nous permet de visualiser les communautés de Louvain, la visualisation suivante : 

![alt](https://user-images.githubusercontent.com/104632559/201882303-1c3cf49f-b5af-4308-9958-5fa4823e8e48.png)


---


## Women’s World Cup 2019

On visualise le graphe : 

![alt](https://user-images.githubusercontent.com/104632559/202009565-40e1925b-01d9-46a1-95e0-80643036a222.svg)

De même pour avoir une meilleur compréhension du graphe on visualise tous les noeuds du graphes : 
![alt](https://user-images.githubusercontent.com/104632559/202011173-ebcd6e1f-8e27-431b-9734-fffdd42230cb.svg)


On observe donc que ce graphe contient des informations sur les huit dernières coupes du monde : 
- Les matchs 
- Les équipes
- Les membres

Il y’a 2 types de nœud pour les équipes : 
- Squad : La composition de l'équipe au moment de la compétition
- Team : La nation représentée par cette équipe

### Requêtes simples
- Afficher les équipes jouant leure première coupe du monde : 
```sql
MATCH (t:Tournament {year: 2019})<-[:PARTICIPATED_IN]-(team)
WITH team, [(team)-[:PARTICIPATED_IN]->(other) WHERE other.year < 2019 | other] AS otherTournaments
WHERE size(otherTournaments) = 0
RETURN team.name 
``` 
Cela donne le csv suivant : 
|team.name|
|---------|
|Chile    |
|Jamaica  |
|South Africa|
|Scotland |

- Finale et gagant des dernières coupes du monde : 
```sql
MATCH (t1:Team)-[p1:PLAYED_IN]-(m:Match)<-[p2:PLAYED_IN]-(t2:Team),
      (m)-[:IN_TOURNAMENT]->(tourn)
WHERE id(t1) < id(t2) AND m.stage = "Final"
RETURN tourn.name AS name, tourn.year AS year,
       t1.name AS team1, t2.name AS team2,
       CASE WHEN p1.score = p2.score
            THEN p1.score + "-" + p2.score + " (" +
                 p1.penaltyScore + "-" + p2.penaltyScore + ")"
            ELSE p1.score + "-" + p2.score
       END AS result,
       (CASE WHEN p1.score > p2.score THEN t1
             WHEN p2.score > p1.score THEN t2
             ELSE
              CASE WHEN p1.penaltyScore > p2.penaltyScore THEN t1
                   ELSE t2 END END).name AS winner
ORDER BY tourn.year
```
qui donne le csv suivant : 

|WCLocation|Year|Team1  |Team2      |result   |winner |
|----------|----|-------|-----------|---------|-------|
|China PR 1991|1991|Norway |USA        |1-2      |USA    |
|Sweden 1995|1995|Germany|Norway     |0-2      |Norway |
|USA 1999  |1999|USA    |China PR   |0-0 (5-4)|USA    |
|USA 2003  |2003|Germany|Sweden     |2-1      |Germany|
|China 2007|2007|Germany|Brazil     |2-0      |Germany|
|Germany 2011|2011|Japan  |USA        |2-2 (3-1)|Japan  |
|Canada 2015|2015|Japan  |USA        |2-5      |USA    |
|France 2019|2019|USA    |Netherlands|2-0      |USA    |

- Requête pour trouver les joueuses les plus efficaces en sortie de banc : 
```sql
MATCH (p:Person)-[:SCORED_GOAL]->(match)<-[:PLAYED_IN {type: "Subbed On"}]-(p)
WITH p, count(*) AS goals
MATCH (p)-[:REPRESENTS]-(team)
RETURN p.name AS Name, team.name AS Country, goals AS Buts
ORDER BY goals DESC
```

qui donne le csv suivant : 

|Name|Country|Buts   |
|----|-------|-------|
|Lisa De Vanna|Australia|4      |
|Aurora Galli|Italy  |3      |
|Martina Mueller|Germany|2      |
|Zhang Ouying|China PR|2      |
|Elena Fomina|Russia |2      |

---

## Gephi

On commence par une première visualisation du graphe Gephi :

![alt](https://user-images.githubusercontent.com/104632559/202017107-7c404d3f-19a8-4d11-a42d-17775651ed52.png)

Puis on essaye de visualiser les aéroport ayant le plus de routes en utilisant le layout PageRank de gephi :

<br>

![alt](https://user-images.githubusercontent.com/104632559/202015918-5831b086-5d5a-41b1-9999-8ff35ff54e21.png)


On remarque que les noeuds présentent des attributs longitude et latitude ce qui nous permet de les représenter sur une carte ce qui donne le résultat suivant :

![alt](https://user-images.githubusercontent.com/104632559/202022122-8e3cbe5d-bb0a-4307-859c-1e927ce0af14.png)

On peut commencer par colorer les noeuds par couleur de pays et par retirer les arêtes entre les graphes : 
![alt](https://user-images.githubusercontent.com/104632559/202030582-dc50e010-6dae-4069-bb39-7a8afa79e170.png)


On remarque une ligne étrange en dessous du graphe qui correspond à des points qui ont des labels qui ne sont pas aéroports. Il y'a 4 types de label ('airport', 'country', 'continent', 'version'). On filtre donc le graphe pour n'afficher que les labels valant 'airport'.

![alt](https://user-images.githubusercontent.com/104632559/202030593-8d12d7eb-5bfd-4747-8cd9-400c77079d23.png)

Cette visualisation par pays est utile mais mais on peut essayer de visualiser le graphe par communauté pour voir si on peut apercevoir une séparation par continent. 
Donc on applique l'algorithme de modularité de Louvain afin de détecter les communautés et on utilise une palette de 9 couleurs afin d'englober toutes les communautés avec une part d'éléments de plus de 2% :

![alt](https://user-images.githubusercontent.com/104632559/202034757-31193e32-25fd-4813-8dca-42205cd4b057.png)

On peut aussi ajouter une visualisation sur la taille des aéroports en fonction de leurs degrés d'arrivants : 

![alt](https://user-images.githubusercontent.com/104632559/202035116-f66a1763-b9c6-4c07-84d2-8838b63d31ce.png)

On obtient alors la visualisation suivante quu nous permet de distinguer les plus gros hubs. 

On peut aussi essayer de faire une visualisation en utilisant l'algorithme PageRank qui donne une visualisation légèrement différentent : 
![alt](https://user-images.githubusercontent.com/104632559/202035547-d87d0913-48b7-46f9-90f2-06dfbb14fe4f.png)


On décide aussi d'afficher une vue mettant en lumières la centralité des composantes : 
![alt](https://user-images.githubusercontent.com/104632559/202036125-b329b6c5-e72b-41fe-8ac3-f046c9d7a1cf.png)

On remarque que les trois dernières images donnent des résultats très similaires. Néanmmoins, l'algorithme de centralité met tout de même bien en avant la centralité des noeuds européens.

Charles Ouazana.
