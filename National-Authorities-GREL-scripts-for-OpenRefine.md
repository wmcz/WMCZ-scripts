- [GREL scripts and corresponding Catmandu calls for extraction of Wikidata-ready data from Czech National Authority files](#grel-scripts-and-corresponding-catmandu-calls-for-extraction-of-wikidata-ready-data-from-czech-national-authority-files)
  * [100abq,500ia7 (Pseudonyms)](#100abq-500ia7--pseudonyms-)
    + [Catmandu](#catmandu)
    + [GREL to exclude pseudonym entries from import](#grel-to-exclude-pseudonym-entries-from-import)
    + [GREL to add pseudonyms of each item as authority IDs](#grel-to-add-pseudonyms-of-each-item-as-authority-ids)
  * [100abq,678a (Czech description)](#100abq-678a--czech-description-)
    + [Catmandu](#catmandu-1)
    + [Python](#python)
  * [100abq (Personal name)](#100abq--personal-name-)
    + [Catmandu](#catmandu-2)
    + [GREL](#grel)
  * [100abq,[gender] (Birth name)](#100abq--gender---birth-name-)
    + [Catmandu](#catmandu-3)
    + [Required datafiles](#required-datafiles)
    + [GREL](#grel-1)
  * [100a (Surname)](#100a--surname-)
    + [Catmandu](#catmandu-4)
    + [Required datafiles](#required-datafiles-1)
    + [GREL](#grel-2)
  * [100d,046f,678a (Date of birth)](#100d-046f-678a--date-of-birth-)
    + [Catmandu](#catmandu-5)
    + [GREL](#grel-3)
  * [100d,046g,678a (Date of death)](#100d-046g-678a--date-of-death-)
    + [Catmandu](#catmandu-6)
    + [GREL](#grel-4)
  * [370ab,678a (Place of birth)](#370ab-678a--place-of-birth-)
    + [Catmandu](#catmandu-7)
    + [Required datafiles](#required-datafiles-2)
    + [GREL](#grel-5)
  * [370ab,678a (Place of death)](#370ab-678a--place-of-death-)
    + [Catmandu](#catmandu-8)
    + [Required datafiles](#required-datafiles-3)
    + [GREL](#grel-6)
  * [374a,678a (Occupation)](#374a-678a--occupation-)
    + [Catmandu](#catmandu-9)
    + [Required datafiles](#required-datafiles-4)
    + [Python](#python-1)
  * [375a,678a (Sex/gender)](#375a-678a--sex-gender-)
    + [Catmandu](#catmandu-10)
    + [GREL](#grel-7)
  * [377a (Languages written)](#377a--languages-written-)
    + [Catmandu](#catmandu-11)
    + [Required datafiles](#required-datafiles-5)
    + [GREL](#grel-8)

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

### 100abq,678a (Czech description)

#### Catmandu

`catmandu convert MARC --type XML --fix data/100abq,678a.fix to CSV --fields "_id,100a,100b,100q,678a" < data/aut.xml > data/output.csv`

fix:
```
do marc_each()
  marc_map(100a,100a)
  marc_map(100b,100b)
  marc_map(100q,100q)
  marc_map(678a,678a)
end
```

#### Python

```
import re
string=re.sub(" ?\(.*?\)","", cells['678a'].value)
f=""
if len(string) > 250 : 
  array = re.split("(.*?[\.] ?(?![a-z 0-9]))",string)
  for x in array:
    if len(f + x) < 250:
       f = f + x
  if f == "":
    array = re.split(",",string)
    for x in array:
      if len(f + x) < 250:
         f = f + x

else:
  f = string

return f
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
(if(isNonBlank(cells['100q'].value),
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
	)).replace(">>","").replace("<<","")
```

### 100abq,[gender] (Birth name)

#### Catmandu
See personal name. You will also need a column with gender, if known (See section "Sex/gender")

#### Required datafiles
* Birth names from Wikidata (safe list of names previously used in NKC data) <a href="https://github.com/wmcz/WMCZ-scripts/blob/main/birth_names.csv">here</a> (can be updated from <a href="https://query.wikidata.org/#select%20distinct%20%3Fname_item%20%3Fname%20%3Fgender%20%3Fconcatenated%20with%20%7B%0A%20%20select%20%3Fname_item%20%3Fname%20where%20%7B%0A%20%20%20%20%3Fitem%20wdt%3AP691%20%5B%5D%20.%0A%20%20%20%20%3Fitem%20p%3AP735%20%5Bps%3AP735%20%3Fname_item%20%3B%20prov%3AwasDerivedFrom%2Fpr%3AP248%20wd%3AQ13550863%20%3B%20wikibase%3Arank%20wikibase%3ANormalRank%20%5D%20.%0A%20%20%20%20%3Fname_item%20wdt%3AP1705%20%3Fname_in_language%20.%20%0A%20%20%20%20bind%28str%28%3Fname_in_language%29%20as%20%3Fname%29%20filter%28%21contains%28%3Fname%2C%22.%22%29%29.%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7D%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7D%20as%20%25names%20where%20%7B%0A%20%20include%20%25names%0A%20%20optional%20%7B%3Fname_item%20p%3AP31%2Fps%3AP31%20wd%3AQ3409032%20.%20bind%28%22unisex%22%20as%20%3Fgender%20%29%20.%7D%0A%20%20optional%20%7B%3Fname_item%20p%3AP31%2Fps%3AP31%20wd%3AQ12308941%20.%20%0A%20%20%20%20%20%20%20%20%20%20%20%20%3Fname_item%20p%3AP31%2Fps%3AP31%20wd%3AQ11879590%20.%0A%20%20%20%20%20%20%20%20%20%20%20%20bind%28%22unisex%22%20as%20%3Fgender%20%29%20.%7D%0A%20%20optional%20%7B%3Fname_item%20p%3AP31%2Fps%3AP31%20wd%3AQ12308941%20.%20bind%28%22male%22%20as%20%3Fgender%20%29%20.%7D%0A%20%20optional%20%7B%3Fname_item%20p%3AP31%2Fps%3AP31%20wd%3AQ11879590%20.%20bind%28%22female%22%20as%20%3Fgender%29%20.%7D%0A%20%20bind%28%20%28if%28%20bound%28%3Fgender%29%20%2Cconcat%28str%28%3Fname%29%2C%22_%22%2Cstr%28%3Fgender%29%29%2Cconcat%28str%28%3Fname%29%2C%22_unisex%22%29%20%29%29%20as%20%3Fconcatenated%20%29%20.%0A%20%20%0A%7D%20group%20by%20%3Fname_item%20%3Fname%20%3Fgender%20%3Fconcatenated">this SPARQL query</a>)

#### GREL
```
forEach(
	with(cells['name'].value.split(" "),
		namearray,
		slice(slice(namearray,0,length(namearray)-1),0,3)),
	name,
	with(
		if(isBlank(cells['gender'].value),
			if(isError(name.cross('birth_names','name').cells['name_item'].value[1]),name.cross('birth_names','name').cells['name_item'].value[0],null),
			(name + "_" + cells['gender'].value.replace("Q6581097","male").replace("Q6581072","female")).cross('birth_names','concatenated').cells['name_item'].value[0]
			),
		gender_specific_version,
		if(isNull(gender_specific_version),(name + "_unisex").cross('birth_names','concatenated').cells['name_item'].value[0],gender_specific_version)
		)
	).toString().replace(/[\[\] ]/,"")
```

### 100a (Surname)

#### Catmandu
See section related to birth name.

#### Required datafiles
* list of surnames (surnames.csv), you can update it from <a href="https://query.wikidata.org/#select%20distinct%20%3Fsurname%20%28sample%28%3Flabel_str%29%20as%20%3Flabel_str%29%20%28sample%28%3Falphabet%29%20as%20%3Falphabet%29%20where%20%7B%0A%20%20%0A%20%20%3Fsurname%20wdt%3AP31%20wd%3AQ101352%20.%0A%20%20minus%20%7B%3Fsurname%20wdt%3AP31%20wd%3AQ4167410%20.%7D%0A%20%20%3Fsurname%20wdt%3AP1705%20%3Flabel%20.%0A%20%20bind%28str%28%3Flabel%29%20as%20%3Flabel_str%29%20.%0A%20%20optional%20%7B%3Fsurname%20wdt%3AP282%20%3Falphabet%20.%20%7D%0A%20%20%0A%7D%20group%20by%20%3Fsurname%20%3Flabel_str%20%3Falphabet">this</a> SPARQL query)

#### GREL

```
forEach(
	with(cells['100a'].value,
		v,
		substring(v,0,indexOf(v,",")
		)
	).split("-"),
	surname,
	strip(surname).cross('surnames','label').cells['surname'].value[0]).join(",")
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
The task is not trivial in GREL, so Python (also natively supported by OpenRefine) is used instead. The task can take up to several hours on a full dataset.

Be sure to also change the paths to files in the script.

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
### 377a (Languages written)

#### Catmandu
`catmandu convert MARC --type XML --fix data/377a.fix to CSV --fields "_id,377a" < data/aut.xml > data/output.csv`

fix:

```
do marc_each()
  marc_map(377a,377a.$append,join:"$")
end
join_field(377a,'|')
```
#### Required datafiles
* <a href="https://github.com/wmcz/WMCZ-scripts/blob/main/jazyky.csv">jazyky</a> - language code table, update from <a href="https://query.wikidata.org/#select%20distinct%20%3Fitem%20%3Fkod%20where%20%7B%0A%0A%20%20%3Fitem%20wdt%3AP219%20%5B%5D%20.%0A%20%20optional%20%7B%3Fitem%20p%3AP219%20%3Fs%20.%20%3Fs%20ps%3AP219%20%3Fkod1%20.%20minus%20%7B%3Fs%20pq%3AP518%20wd%3AQ1631107%20.%20%7D%20%7D%0A%20%20optional%20%7B%3Fitem%20p%3AP219%20%5Bps%3AP219%20%3Fkod2%20%3B%20pq%3AP518%20wd%3AQ1631107%20%5D%20.%7D%0A%20%20bind%28coalesce%28%3Fkod2%2C%3Fkod1%29%20as%20%3Fkod%29%20.%0A%0A%7D">this</a> query

#### GREL

```
forEach(
	filter(
		cells['377a'].value.replace("|","$").split("$").uniques(),
		x,x!="mul"
		),
	v,
	v.cross('jazyky','kod').cells['item'].value[0]).join(",")
```

### 024 (ORCID, ISNI, Wikidata QID)

#### Catmandu
`catmandu convert MARC --type XML --fix data/024.fix to CSV --fields "_id,024a2" < data/aut.xml > data/output.csv`

```
do marc_each()
  marc_map(024a2,024a2.$append,join:"$")
end
join_field(024a2,'|')
```

#### GREL

ISNI:

```
filter(cells['0247a2'].value.split("|"),v,v.contains("isni"))[0].strip().chomp("$isni").match(/^([0-9]{16})$/)[0].splitByLengths(4,4,4,4).join(" ")
```

ORCID:

```
filter(cells['0247a2'].value.split("|"),v,v.contains("orcid"))[0].strip().chomp("$orcid").match(/^(0000-000(1\-[5-9]|2\-[0-9]|3\-[0-4])\d{3}\-\d{3}[\dX])$/)[0]
```

QID:

```
filter(cells['0247a2'].value.split("|"),v,v.contains("wikidata"))[0].strip().chomp("$wikidata").match(/^(Q[0-9]{1,10})$/)[0]

```

Always check that the QID has not been deprecated in Wikidata (source data from <a href="https://query.wikidata.org/#select%20%3Fitem%20%3Fnkcr%20where%20%7B%0A%0A%20%20%3Fitem%20p%3AP691%20%5Bps%3AP691%20%3Fnkcr%20%3B%20wikibase%3Arank%20%3Frank%20%5D%20filter%28%3Frank%20%3D%20wikibase%3ADeprecatedRank%29%20.%0A%0A%7D">this</a> query): 

```
if(cells['_id'].value == cells['wikidata'].value.cross('deprecated nkcr','item').cells['nkcr'].value[0],"do not import",null)
```
