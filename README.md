### How  ?

1. New setup 
	```bash
	cd <blog-dir>
	sudo gem install jekyll bundler
	bundle install
	```
2. Start locally 
	```bash
		bundle exec jekyll serve
	```
3. How to add new post ?
	-  Add  new markdown in folder `_posts`
	- Format should be like  `2016-05-21-<post_title>.md`
	
4. How to build ?
	- Build
		```bash
			bundle exec jekyll build
		```
	- Above will generate required code in `_site` folder .
	- Commit generated folder files into `master` branch .
	- GitHub will automatically deploy it for you .
	
That's it :)
	