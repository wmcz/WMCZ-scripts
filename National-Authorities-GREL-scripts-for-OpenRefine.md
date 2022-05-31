## GREL scripts and corresponding Catmandu calls for extraction of Wikidata-ready data from Czech National Authority files

  * [100abq (Personal name)](#100abq--personal-name-)
    + [GREL](#grel)
    + [Catmandu](#catmandu)
  * [100d,046f,678a (Date of birth)](#100d-046f-678a--date-of-birth-)
    + [GREL](#grel-1)
    + [Catmandu](#catmandu-1)
  * [100d,046g,678a (Date of death)](#100d-046g-678a--date-of-death-)
    + [GREL](#grel-2)
    + [Catmandu](#catmandu-2)
  * [370ab,678a (Place of birth)](#370ab-678a--place-of-birth-)
    + [Required datafiles](#required-datafiles)
    + [GREL](#grel-3)
    + [Catmandu](#catmandu-3)
  * [370ab,678a (Place of death)](#370ab-678a--place-of-death-)
    + [Required datafiles](#required-datafiles-1)
    + [GREL](#grel-4)
    + [Catmandu](#catmandu-4)
  * [374a,678a (Occupation)](#374a-678a--occupation-)
    + [Required datafiles](#required-datafiles-2)
    + [Python](#python)
    + [Catmandu](#catmandu-5)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>

* Catmandu script = to extract information from MARC XML dumps
* OpenRefine GREL or Python code = to transform values for Wikidata

### 100abq,500ia7 (Pseudonyms)
Czech authority files handle pseudonyms as separate entries while Wikidata keep them generally in one item.
#### Catmandu

`catmandu convert MARC --type XML --fix data/100abq,500ia7.fix to CSV --fields "_id,100a,100b,100q,500ia7" < data/aut.xml > data/output.csv`

fix:

```
do marc_each()
  marc_map(100a,100a)
  marc_map(100b,100b)
  marc_map(100q,100q)
  marc_map(500ia7,500ia7.$append,join:"$") 
  
end

join_field(500ia7,'|')
```

#### GREL to exclude pseudonym entries from import

1) extract pseudonym authority IDs:

Create new column "pseudonyms"

```
forEach(filter(cells['500ia7'].value.split("|"),v,contains(v,/Pseudonym\:.*/)),v,v.match(/.*?Pseudonym\:\$[^\$]+\$(.*?)$/)[0]).join(',')

```

2) exclude pseudonym rows from import:

```
with(cells['_id'].value.cross('','pseudonyms').cells['pseudonyms'].value[0],findpseudo,if(isNull(findpseudo),null,"do not import"))
```

#### GREL to add pseudonyms of each item as authority IDs

```

forEach(filter(cells['500ia7'].value.split("|"),v,contains(v,/Pseudonym\:.*/)),v,v.split("$").slice(1,3).reverse().join("$").chomp(",")).join("|")

```

### 100abq (Personal name)

#### Catmandu

`catmandu convert MARC --type XML --fix data/100abq.fix to CSV --fields "_id,100a,100b,100q" < data/aut.xml > data/output.csv`

fix:
```
do marc_each()
  marc_map(100a,100a)
  marc_map(100b,100b)
  marc_map(100q,100q)
end
```
#### GREL
```
  if(isNonBlank(cells['100q'].value),
	cells['100q'].value.chomp(",").strip().replace("(","").replace(")","") + " " + cells['100a'].value.chomp(",").split(",").get(0).strip(),
	if(cells['100a'].value.chomp(",").split(",").length() == 3,
		cells['100a'].value.chomp(",").split(",").get(1).strip() + " " + cells['100a'].value.chomp(",").split(",").get(2).strip() + " " + cells['100a'].value.chomp(",").split(",").get(0).strip(),
		if(cells['100a'].value.chomp(",").split(",").length() == 2,
			cells['100a'].value.chomp(",").split(",").get(1).strip() + " " + cells['100a'].value.chomp(",").split(",").get(0).strip(),
			cells['100a'].value.chomp(",").strip()
			)
		)
	)	
+ if(isNonBlank(cells['100b'].value),
	" " + cells['100b'].value.strip().chomp(","),
	""
	)
```
  
### 100d,046f,678a (Date of birth)

#### Catmandu

`catmandu convert MARC --type XML --fix data/100d,046fg,678a.fix to CSV --fields "_id,100d,046f,046g,678a" < data/aut.xml > data/output.csv`

fix:
```
do marc_each()
  marc_map(100d,100d)
  marc_map(046f,046f)
  marc_map(046g,046g)  
  marc_map(678a,678a.$append)
end
join_field(678a,'|')
```

#### GREL

```
with([
	with(if (cells['046f'].value.length() == 8,
		cells['046f'].value.splitByLengths(4,2,2).join("-"),
		if (cells['046f'].value.length() == 6,
			cells['046f'].value.splitByLengths(4,2).join("-"),
			if (cells['046f'].value.length() == 4,cells['046f'].value,"")
			)
		).match(/(^[0-9]{2}\.[0-9]{2}\.[0-9]{3,4}$|^[0-9]{3,4}$|^[0-9]{0,2}\.[0-9]{3,4}$)/)[0],
	part1,if(isError(part1),"",part1))
	,
	with(cells['678a'].value.match(/.*?[Nn]aro[a-zíýá]*\.? ?(se )?(ve? )?(roku |roce |r\. ?|\:? ?)?([0-9\. ]*(lednu|ledna|únoru|února|březnu|března|dubnu|dubna|květnu|května|červnu|června|červenci|července|srpnu|srpna|září|říjnu|října|listopadu|prosinci|prosince)? ?(roku |r\.)?[^a-zá-žA-ZÁ-Ž,\-\(]*).*/)[3].
		strip().chomp(".").chomp(",").
		replace("ledna","1.").replace("lednu","1.").
		replace("února","2.").replace("únoru","2.").
		replace("března","3.").replace("březnu","3.").
		replace("dubna","4.").replace("dubnu","4.").
		replace("května","5.").replace("květnu","5.").
		replace("června","6.").replace("červnu","6.").
		replace("července","7.").replace("červenci","7.").
		replace("srpna","8.").replace("srpnu","8.").
		replace("září","9.").
		replace("října","10.").replace("říjnu","10.").
		replace("listopadu","11.").
		replace("prosince","12.").replace("prosinci","12.").
		replace("roku","").replace("r.","").replace(" ","").
		match(/(^[0-9]{0,2}\.?[0-9]{0,2}\.?[0-9]{3,4}$)/)[0].
		replace(/^([1-9]\.)/,"0$1").replace(/\.([1-9]\.)/,"\.0$1").replace(/\.([0-9]{3}$)/,"\.0$1").
		match(/(^[0-9]{2}\.[0-9]{2}\.[0-9]{3,4}$|^[0-9]{3,4}$|^[0-9]{0,2}\.[0-9]{3,4}$)/)[0].
		split(".").reverse().join("-"),
	part2,if(isError(part2),"",part2))
	,
	with(cells['100d'].value.match(/^([0-9]{3,4} ?(leden|únor|březen|duben|květen|duben|červenec|červen|srpen|září|říjen|listopad|prosinec)? ?[0-9]{0,2}\.?) ?\-?.*/)[0].
		replace("ledna","1.").replace("leden","1.").
		replace("února","2.").replace("únor","2.").
		replace("března","3.").replace("březen","3.").
		replace("dubna","4.").replace("duben","4.").
		replace("května","5.").replace("květen","5.").
		replace("července","7.").replace("červenec","7.").
		replace("června","6.").replace("červen","6.").
		replace("srpna","8.").replace("srpen","8.").
		replace("září","9.").
		replace("října","10.").replace("říjen","10.").
		replace("listopadu","11.").replace("listopad","11.").
		replace("prosince","12.").replace("prosinec","12.").
		replace(/^([0-9]{3,4} )/,"$1-").replace(" ","").replace(/\.$/,"").replace(".","-").
		replace(/^([0-9]{3}\-)/,"0$1").replace(/\-([0-9]\-)/,"\-0$1").replace(/\-([0-9]$)/,"\-0$1"),
	part3,if(isError(part3),"",part3))
].join(",").split(','), a, filter(a, v, v.length() == forEach(a, x, x.length()).sort()[-1]))[0]
```

### 100d,046g,678a (Date of death)
#### Catmandu

`catmandu convert MARC --type XML --fix data/100d,046fg,678a.fix to CSV --fields "_id,100d,046f,046g,678a" < data/aut.xml > data/output.csv`

fix:
```
do marc_each()
  marc_map(100d,100d)
  marc_map(046f,046f)
  marc_map(046g,046g)  
  marc_map(678a,678a.$append)
end
join_field(678a,'|')
```

#### GREL
```
with([
	with(if (cells['046g'].value.length() == 8,
		cells['046g'].value.splitByLengths(4,2,2).join("-"),
		if (cells['046g'].value.length() == 6,
			cells['046g'].value.splitByLengths(4,2).join("-"),
			if (cells['046g'].value.length() == 4,cells['046g'].value,"")
			)
		).match(/(^[0-9]{2}\.[0-9]{2}\.[0-9]{3,4}$|^[0-9]{3,4}$|^[0-9]{0,2}\.[0-9]{3,4}$)/)[0],
	part1,if(isError(part1),"",part1))
	,
	with(cells['678a'].value.match(/.*?([Zz]e|[Uu])mř[a-zíýá]*\.? ?(ve? )?(roku |roce |r\. ?|\:? ?)?([0-9\. ]*(lednu|ledna|únoru|února|březnu|března|dubnu|dubna|květnu|května|červnu|června|červenci|července|srpnu|srpna|září|říjnu|října|listopadu|prosinci|prosince)? ?(roku |r\.)?[^a-zá-žA-ZÁ-Ž,\-\(]*).*/)[3].
		strip().chomp(".").chomp(",").
		replace("ledna","1.").replace("lednu","1.").
		replace("února","2.").replace("únoru","2.").
		replace("března","3.").replace("březnu","3.").
		replace("dubna","4.").replace("dubnu","4.").
		replace("května","5.").replace("květnu","5.").
		replace("června","6.").replace("červnu","6.").
		replace("července","7.").replace("červenci","7.").
		replace("srpna","8.").replace("srpnu","8.").
		replace("září","9.").
		replace("října","10.").replace("říjnu","10.").
		replace("listopadu","11.").
		replace("prosince","12.").replace("prosinci","12.").
		replace("roku","").replace("r.","").replace(" ","").
		match(/(^[0-9]{0,2}\.?[0-9]{0,2}\.?[0-9]{3,4}$)/)[0].
		replace(/^([1-9]\.)/,"0$1").replace(/\.([1-9]\.)/,"\.0$1").replace(/\.([0-9]{3}$)/,"\.0$1").
		match(/(^[0-9]{2}\.[0-9]{2}\.[0-9]{3,4}$|^[0-9]{3,4}$|^[0-9]{0,2}\.[0-9]{3,4}$)/)[0].
		split(".").reverse().join("-"),
	part2,if(isError(part2),"",part2))
	,
	with(cells['100d'].value.match(/.*?\-([0-9]{3,4} ?(leden|únor|březen|duben|květen|duben|červenec|červen|srpen|září|říjen|listopad|prosinec)? ?[0-9]{0,2}\.?).*/)[0].
		replace("ledna","1.").replace("leden","1.").
		replace("února","2.").replace("únor","2.").
		replace("března","3.").replace("březen","3.").
		replace("dubna","4.").replace("duben","4.").
		replace("května","5.").replace("květen","5.").
		replace("července","7.").replace("červenec","7.").
		replace("června","6.").replace("červen","6.").
		replace("srpna","8.").replace("srpen","8.").
		replace("září","9.").
		replace("října","10.").replace("říjen","10.").
		replace("listopadu","11.").replace("listopad","11.").
		replace("prosince","12.").replace("prosinec","12.").
		replace(/^([0-9]{3,4} )/,"$1-").replace(" ","").replace(/\.$/,"").replace(".","-").
		replace(/^([0-9]{3}\-)/,"0$1").replace(/\-([0-9]\-)/,"\-0$1").replace(/\-([0-9]$)/,"\-0$1"),
	part3,if(isError(part3),"",part3))
].join(",").split(','), a, filter(a, v, v.length() == forEach(a, x, x.length()).sort()[-1]))[0]
```

### 370ab,678a (Place of birth)
#### Catmandu

`catmandu convert MARC --type XML --fix data/370ab,678a.fix to CSV --fields "_id,370a,370b,678a" < data/aut.xml > data/output.csv`

fix:
```
do marc_each()
  marc_map(370a,370a)
  marc_map(370b,370b)
  marc_map(678a,678a)
end
```

#### Required datafiles

* <a href="https://github.com/wmcz/WMCZ-scripts/blob/main/geoauthorities.csv">geoauthorities</a>
* <a href="https://github.com/wmcz/WMCZ-scripts/blob/main/NKCR-QID-convert-table.csv">NKCR-QID convert table</a> (from <a href="https://query.wikidata.org/#select%20%3Fitem%20%3Fnkcr%20where%20%7B%0A%0A%20%20%3Fitem%20p%3AP691%20%5Bps%3AP691%20%3Fnkcr%20%3B%20wikibase%3Arank%20%3Frank%20%5D%20filter%28%3Frank%20%21%3D%20wikibase%3ADeprecatedRank%29%20.%0A%0A%7D">this</a> query)
* <a href="https://github.com/wmcz/WMCZ-scripts/blob/main/mista.csv">mista</a>

#### GREL

```
coalesce(
	with((cells['370a'].value.
		match(/(^.*?)\, (.*)$/).join(" (") + ")").
		cross('geoauthorities','151a').cells['_id'].value[0].cross('NKCR-QID convert table','nkcr').
		cells['item'].value[0]
	,part1,if(isError(part1),null,part1))
	,
	with(cells['678a'].value.
		replace(" n."," nad").
		replace(" p."," pod").
		match(/[Nn]ar[a-zíýá]*\.? ?(se )?(asi )?(v lednu|v únoru|v březnu|v dubnu|v květnu|v červnu|v červenci|v srpnu|v září|v říjnu|v listopadu|v prosinci|v roce|roku|kolem roku|v r\.)? ?([^a-z]*) (ve?|na) ([a-zA-ZÀ-ž ]*)(\.|,|-|;)*.*/)[5].
		strip().
		cross('mista','place').cells['QID'].value[0]
	,part2,if(isError(part2),null,part2))
)
```

### 370ab,678a (Place of death)

#### Catmandu
See section: place of birth.

#### Required datafiles
See section: place of birth.

#### GREL
```
coalesce(
	with((cells['370b'].value.
		match(/(^.*?)\, (.*)$/).join(" (") + ")").
		cross('geoauthorities','151a').cells['_id'].value[0].cross('NKCR-QID convert table','nkcr').
		cells['item'].value[0]
	,part1,if(isError(part1),null,part1))
	,
	with(cells['678a'].value.
		replace(" n."," nad").
		replace(" p."," pod").
		match(/.*?([Zz]e|[Uu])mř[a-zíýá]*\.? ?(asi )?(v lednu|v únoru|v březnu|v dubnu|v květnu|v červnu|v červenci|v srpnu|v září|v říjnu|v listopadu|v prosinci|v roce|roku|kolem roku|v r\.)?  ?([^a-z]*) (ve?|na) ([a-zA-ZÀ-ž ]*)(\.|,|-|;)*.*/)[5].
		strip().
		cross('mista','place').cells['QID'].value[0]
	,part2,if(isError(part2),null,part2))
	,
	if(cells['678a'].value.replace(" n."," nad").replace(" p."," pod").match(/.*?([Zz]emřel|[Uu]mřel|[Pp]opraven)(.*?[^0-9])[\.,].*/)[1].contains("tamtéž"),
		with(cells['678a'].value.
			replace(" n."," nad").
			replace(" p."," pod").
			match(/[Nn]ar[a-zíýá]*\.? ?(se )?(asi )?(v lednu|v únoru|v březnu|v dubnu|v květnu|v červnu|v červenci|v srpnu|v září|v říjnu|v listopadu|v prosinci|v roce|roku|kolem roku|v r\.)? ?([^a-z]*) (ve?|na) ([a-zA-ZÀ-ž ]*)(\.|,|-|;)*.*/)[5].
			strip().
			cross('mista','place').cells['QID'].value[0]
			,part3,if(isError(part3),null,part3)),
		null)
)
```

### 374a,678a (Occupation)

#### Catmandu

`catmandu convert MARC --type XML --fix data/374a.fix to CSV --fields "_id,374a,678a" < data/aut.xml > data/output.csv`

fix:

```
do marc_each()
  marc_map(374a,374a.$append,join:"|")
  marc_map(678a,678a)
end

join_field(374a,'|')
```

#### Required datafiles

* <a href="https://github.com/wmcz/WMCZ-scripts/blob/main/povolani.csv">povolani</a> (occupations as they appear in field 678a)
* <a href="https://github.com/wmcz/WMCZ-scripts/blob/main/povolani2.csv">povolani2</a> (occupations as they appear in field 374a)

#### Python
The task is not trivial in GREL, so Python (also natively supported by OpenRefine) is used instead.
Be sure to change the paths to files in the script.

```
stopwords = {}

with open(r"/Users/vojte\Documents/povolani.csv",'r') as f:
 for name in f:
  stopwords[name.decode('utf8').split(";")[0]] = name.rstrip().split(";")[1]

with open(r"/Users/vojte\Documents/povolani2.csv",'r') as f:
 for name in f:
  stopwords[name.decode('utf8').split(";")[0]] = name.rstrip().split(";")[1]

add = []

try:
 for x in cells['374a'].value.split('|'):
  if x.lower() in stopwords:
   if (stopwords[x.lower()] not in add):
    add.append(stopwords[x.lower()])
except KeyError:
  pass

try:
 for x in cells['678a'].value.replace(',',' ').replace('.',' ').split(' '):
  if x.lower() in stopwords:
   add.append(stopwords[x.lower()])
except KeyError:
  pass

result = []
[result.append(x) for x in add if x not in result]

joined = ",".join(result)
return joined 
```

### 375a,678a (Sex/gender)

#### Catmandu
`catmandu convert MARC --type XML --fix data/375a.fix to CSV --fields "_id,375a,678a" < data/aut.xml > data/output.csv`

fix:

```
do marc_each()
  marc_map(375a,375a.$append,join:"|")
  marc_map(678a,678a)
end

join_field(375a,'|')
```

#### GREL

```
with(cells['375a'].value,
	option1,
	if(option1.contains(/^(muž|žena)$/),
		if(option1.contains(/^(muž)$/),
			"Q6581097",
			"Q6581072"
			),
		with(cells['678a'].value,
			option2,
			if(option2.contains(/^(Narozen |Německý |Americký | Francouzský |Autor | Britský | Ruský |Polský |Italský |Slovenský |Rakouský |Španělský |Anglický ).*$/),
				"Q6581097",
				if(option2.contains(/^(Narozena |Německá |Americká | Francouzská |Autorka | Britská | Ruská |Polská |Italská |Slovenská |Rakouská |Španělská |Anglická ).*$/),
					"Q6581072",
					null
					)
				)
			)
		)
)
```
