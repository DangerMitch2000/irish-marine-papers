# Irish Marine Science Papers

An open, auto-updating catalogue of peer-reviewed Irish marine research.

| **Live demo** | https://DangerMitch2000.github.io/irish-marine-papers/ |
|---------------|-----------------------------------------------------|

## How to contribute
1. Fork the repo  
2. Enable GitHub Pages (source = “Deploy from branch → /docs”)  
3. Edit `crawler/fetch.py` if you want to tweak keywords or add new sources  
4. Open a PR—GitHub Actions will test the crawler automatically

## Local test
```bash
pip install requests
python crawler/fetch.py
# open docs/index.html in your browser
