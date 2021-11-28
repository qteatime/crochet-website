run:
	bundle exec jekyll serve --drafts

build:
	rm -rf _site
	bundle exec jekyll build
	git submodule update
	rm -rf _docs/_build
	cd _docs && make html
	mv _docs/_build/html _site/docs
	mkdir _site/_static
	cp -R _site/docs/_static/api _site/_static
	
