# Bank Link Webcrawler

This is a web scraper containing various spiders written in Python using the Scrapy framework for purposes of crawling bank websites for information. Spiders take in a .csv file in various specified formats as an input, and outputs various results into the console or a .csv or .json format.

## Spiders
This spiders written for this project are the following:
### `links`
`links` is a webcrawler that simply scrapes the links from a list of designated websites, the links' response codes(including non-200 response codes), their originating start URLs, and the name of the website itself.

### `linksExtract`
`linksExtract` is a modified version of the `links` spider that contains additional Splash integration for JavaScript rendered content. All spiders below include additional Splash integration. See below sections on details for running a local Splash server for these requests to succeed.

### `linksSearch`
`linksSearch` crawls bank websites and find target links. The domain and path of where the link was found is provided, as well as a preset optional description provided by the user in the list of target links to search for.

### `spreadsheetFill`
A spider that takes in a spreadsheet in a specific format, pulls the relevant information from the provided URLs, and fills the remainder of the spreadsheet with the needed information. Specific use case, not very applicable to other circumstances.

### `prodLinks`
A spider that detects specific links on bank websites and lists the location of the link through outputting the domain and path the link was found on.

## Docker Container (WIP)
- A working Dockerfile and accompanying docker-compose.yml has been added for containerization of running spiders and generating output. Currently it is working and tested only for the `linksExtract` spider.

## Usage
### Spiders
- Spiders run in a virtual environment on a Linux machine using `virtualenv`.
   - Can be run on Windows and MacOS machines as well, consult Scrapy documentation for setup.
- From the virtual environment in the root of the project (e.g. `/bankcrawler` in this case)
you can run `scrapy crawl [SPIDER_NAME] -o [OUTPUT_FILE_NAME].[FILE_EXTENSION]`
- For example, to run a spider called `links`, do `scrapy crawl links` 
- If you wanted to output to a `.csv`, use the `-o` flag and do `scrapy crawl links -o links.csv`
- Various settings pertaining to crawl speed, output logging, as well as controlling which Scrapy middlewares the spiders use can be found and configured in the settings.py file.
### Splash
- For any spider that uses Splash requests, you must connect the spider to a Splash server before running the spider.
- You must specify the Splash server in the form of a URL with the `SPLASH_URL` in your spider(`settings.py`).
- The easiest way to launch a Splash server is through a Docker container.
   - You can launch a detached container with the following command: `docker run -d -p 8050:8050 scrapinghub/splash`
### Docker
- A docker image for a spider can be generated by editing the Dockerfile to your specific needs, and running `docker build -t [IMAGE_NAME] .`
- In order to run the spider in a container along with Splash, use `docker-compose up` to use the docker-compose.yml file's configuration to launch the spider image along with the required local Splash server. 
- You can retrieve the files generated by running the spider with the `docker cp` command, which copies files from a container to your host machine.
   - In order to do so, find the terminated spider container using `docker ps -a`, and note the first two characters of the container id.
   - Use `docker cp [FIRST_TWO_CHARS]:/usr/src/app .` to get the entire project folder, in which spider logs can be found under `/logs` and .csv output can be found under `/csv`. 

For more detailed usage, please refer to the [Scrapy documentation](https://docs.scrapy.org/en/latest/index.html).

## Troubleshooting
- If your log has messages that say "Connection refused", you most likely do not have the Splash server running on a spider that utlizes SplashRequests. To do so, install Docker if you haven't already, and run `docker run -p 8050:8050 scrapinghub/splash`.

## Required Modules and Dependencies
- Python 3.6+
- Scrapy 1.8+
- Scrapy Splash 0.7.2+
- BeautifulSoup4 4.8.2+
- tldextract