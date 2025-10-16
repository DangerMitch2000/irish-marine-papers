#!/usr/bin/env python3
"""
Fetch recent Irish-marine-science papers from CrossRef and PubMed
and write a single JSON file docs/papers.json
"""
import json, datetime, requests, xml.etree.ElementTree as ET

MARINE_KEYWORDS = ("marine","ocean","sea","coastal","estuar","benthic","plankton","fish","shellfish","seaweed","coral","whale","dolphin","seal")
IRISH_AFFILS     = ("ireland","irish","dublin","cork","galway","limerick","maynooth","ulster","trinity college","ucc","ucd","nuig","nuim")

def crossref():
    """Return list of dicts with title, authors, journal, year, doi, url"""
    rows, offset = [], 0
    while offset < 2000:                       # pagination
        url = ("https://api.crossref.org/works?"
               "filter=type:journal-article,from-pub-date:2010-01-01&"
               "rows=1000&offset={}".format(offset))
        data = requests.get(url, timeout=60).json()["message"]["items"]
        if not data:
            break
        for item in data:
            affils = " ".join(a.get("name","") for a in item.get("author",[]))
            title  = " ".join(item.get("title",[""])).lower()
            if any(k in title+affils for k in MARINE_KEYWORDS) and any(k in affils.lower() for k in IRISH_AFFILS):
                rows.append({
                    "title":   item["title"][0],
                    "author":  ", ".join(a.get("family","") for a in item.get("author",[])),
                    "journal": item["container-title"][0] if item.get("container-title") else "",
                    "year":    item["published"]["date-parts"][0][0],
                    "doi":     item["DOI"],
                    "url":     "https://doi.org/"+item["DOI"]
                })
        offset += 1000
    return rows

def pubmed():
    """Same idea via E-utilities"""
    base   = "https://eutils.ncbi.nlm.nih.gov/entrez/eutils/"
    search = base + "esearch.fcgi?db=pubmed&retmax=10000&term=" + \
             "(marine OR ocean OR coastal OR fish) AND (Ireland OR Irish[ad])"
    ids    = requests.get(search).text
    ids    = [i.text for i in ET.fromstring(ids).findall(".//Id")]
    if not ids:
        return []
    fetch  = base + "efetch.fcgi?db=pubmed&retmode=xml&id="+",".join(ids)
    xml    = requests.get(fetch).text
    rows   = []
    for art in ET.fromstring(xml).findall(".//PubmedArticle"):
        title = art.findtext(".//ArticleTitle") or ""
        jour  = art.findtext(".//Title") or ""
        year  = art.findtext(".//PubDate/Year") or "0"
        pmid  = art.findtext(".//PMID") or ""
        if any(k in title.lower() for k in MARINE_KEYWORDS):
            rows.append({
                "title":title, "journal":jour, "year":int(year),
                "url":"https://pubmed.ncbi.nlm.nih.gov/"+pmid, "doi":"", "author":""
            })
    return rows

def main():
    papers = crossref() + pubmed()
    # de-duplicate by lowercase title
    seen = set()
    uniq = []
    for p in sorted(papers, key=lambda x: x["year"], reverse=True):
        if p["title"].lower() not in seen:
            uniq.append(p)
            seen.add(p["title"].lower())
    with open("docs/papers.json","w",encoding="utf-8") as f:
        json.dump({"lastUpdate":str(datetime.date.today()), "papers":uniq}, f, indent=2)
    print("Wrote {} papers".format(len(uniq)))

if __name__ == "__main__":
    main()
