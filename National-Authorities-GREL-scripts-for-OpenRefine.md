<h2>GREL scripts and corresponding Catmandu calls for extraction of Wikidata-ready data from Czech National Authority files</h2>

* Catmandu call = to extract information from MARC XML dumps
* OpenRefine GREL code = to transform values for Wikidata

<h3>Fields 100abq (Personal name) GREL</h3>
<h4>GREL</h4>
<code>
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
</code>

<h4>Catmandu</h4>

<code>catmandu convert MARC --type XML --fix data/100abq.fix to CSV --fields "_id,100a,100b,100q" < data/aut.xml > data/output.csv</code>

fix:
<code>
do marc_each()
  marc_map(100a,100a)
  marc_map(100b,100b)
  marc_map(100q,100q)
end
</code>
  
<h3>100d,046f,678a (Date of birth)</h3>
  
<h4>GREL</h4>

<code>with(
[if (cells['046f'].value.length() == 8,
	cells['046f'].value.splitByLengths(4,2,2).join("-"),
	if (cells['046f'].value.length() == 6,
		cells['046f'].value.splitByLengths(4,2).join("-"),
		if (cells['046f'].value.length() == 4,cells['046f'].value,"")
		)
	).match(/(^[0-9]{2}\.[0-9]{2}\.[0-9]{3,4}$|^[0-9]{3,4}$|^[0-9]{0,2}\.[0-9]{3,4}$)/)[0]
,
cells['678a'].value.match(/.*?[Nn]aro[a-zíýá]*\.? ?(se )?(asi |kolem )?(ve? )?(roku |roce |r\. ?|\:? ?)?([0-9\. ]*(lednu|ledna|únoru|února|březnu|března|dubnu|dubna|květnu|května|červnu|června|červenci|července|srpnu|srpna|září|říjnu|října|listopadu|prosinci|prosince)? ?(roku |r\.)?[^a-zá-žA-ZÁ-Ž,\-\(]*).*/)[4].
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
split(".").reverse().join("-")
,
cells['100d'].value.match(/.*?([0-9]{3,4} ?(leden|únor|březen|duben|květen|duben|červenec|červen|srpen|září|říjen|listopad|prosinec)? ?[0-9]{0,2}\.?)\ ?-.*/)[0].
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
replace(/^([0-9]{3}\-)/,"0$1").replace(/\-([0-9]\-)/,"\-0$1").replace(/\-([0-9]$)/,"\-0$1")
].
join(",").split(','), a, filter(a, v, v.length() == forEach(a, x, x.length()).sort()[-1]))[0]</code>

<h4>Catmandu</h4>

<code>catmandu convert MARC --type XML --fix data/100d,046fg,678a.fix to CSV --fields "_id,100d,046f,046g,678a" < data/aut.xml > data/output.csv
</code>

fix:
<code>
do marc_each()
  marc_map(100d,100d)
  marc_map(046f,046f)
  marc_map(046g,046g)  
  marc_map(678a,678a.$append)
end
join_field(678a,'|')
</code>

