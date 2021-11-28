run:
	bundle exec jekyll serve --drafts

build:
	git submodule update
	rm -rf _site
	bundle exec jekyll build
	rm -rf _docs/_build
	cd _docs && make html
	mv _docs/_build/html _site/docs

