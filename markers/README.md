Place AR.js NFT marker files in this folder with the base name "article":

- article.fset
- article.fset3
- article.iset

These files are generated from your printed news article image using the AR.js NFT Creator.

Current status:

- This repository now includes a working sample NFT set in article.fset, article.fset3, and article.iset.
- Use markers/article-reference.jpeg as the printable target image for immediate testing.

Deployment checklist:

1. Commit and push all three marker files to GitHub.
2. Verify they are reachable in browser:
	- /markers/article.fset
	- /markers/article.fset3
	- /markers/article.iset
3. If your site is served from a subpath, open index with optional override:
	- ?marker=/your-subpath/markers/article

To switch back to your own article marker:

1. Generate your own NFT files from your printed article image.
2. Replace article.fset, article.fset3, and article.iset.
3. Keep the same base name article so index.html works without changes.
