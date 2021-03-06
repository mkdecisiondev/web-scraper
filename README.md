# Bank Link Webcrawler
This is a web scraper containing various spiders written in Python using the Scrapy framework for purposes of crawling bank websites for information. Spiders take in a .csv file in various specified formats as an input, and outputs various results specified by the various spiders into the console or a .csv or .json format.

## Spiders
Unless otherwise specified, all spiders take in a spreadsheet in the form of a .csv format. Spiders written for this project are the following:
### `links`
`links` is a webcrawler that simply scrapes the links from a list of designated websites (domain/path), the links' response codes(including non-200 response codes), and their originating start URLs.

### `linksExtract`
`linksExtract` is a modified version of the `links` spider that contains additional Splash integration for JavaScript rendered content. All spiders below this one requires Splash to run. See below sections on details for running a local Splash server for these requests to succeed.

### `linksSearch`
`linksSearch` crawls bank websites in order to find target links specified in a additional csv file. The domain and path of where the link was found is provided, as well as a preset optional description provided by the user in the list of target links to search for.

### `spreadsheetFill`
A spider that takes in a spreadsheet in a specific format, pulls the relevant information from the provided URLs, and fills the remainder of the spreadsheet with the needed information. Output in spider must be adjusted for your own specific use case.

### `prodLinks`
A spider that detects MK production and alpha links on bank websites, and lists the location of the link through outputting the domain and path of the bank website the link was found on.

## Docker Container (WIP)
- A working Dockerfile and accompanying docker-compose.yml has been added for containerization of running spiders and generating output. Currently it is working and tested only for the `linksExtract` spider.

## Serverless implementation (WIP)
- Currently experimenting with the Serverless framework to deploy a containerized spider on AWS using ECS and Fargate to manage containers with spiders running inside of them.

## Usage
### Spiders
- Spiders run in a virtual environment on a Linux machine using `virtualenv`.
   - Can be run on Windows and MacOS machines as well, consult Scrapy documentation for setup.
- From the virtual environment in the root of the project (e.g. `/bankcrawler` in this case)
you can run `scrapy crawl [SPIDER_NAME] | -o [OUTPUT_FILE_NAME].[FILE_EXTENSION]`
- For example, to run a spider called `links`, do `scrapy crawl links` 
- If you wanted to output to a `.csv`, use the `-o` flag and do `scrapy crawl links -o links.csv`
- Various settings pertaining to crawl speed, output logging, as well as controlling which Scrapy middlewares the spiders use can be found and configured in the `settings.py` file.

### Splash
- For any spider that uses Splash requests, you must connect the spider to a Splash server before running the spider.
- You must specify the Splash server in the form of a URL with the `SPLASH_URL` in your spider(`settings.py`).
- The easiest way to launch and connect a Splash server is through a locally hosted Docker container.
   - Prerequisites include installing Docker on your machine, along with the `scrapy_splash` Python package.
   - You can launch a detached container with the following command: `docker run -d -p 8050:8050 scrapinghub/splash`
   - If you're not familiar with Docker, the above command calls docker to run a container on the current machine using an image pulled from the public Docker repository `scrapinghub/splash`. The `-d` flag specifies the container to run in a "detached" mode, and the `-p` flag maps the ports from the container to the local machine so your spider can connect to the container with the Splash server.
### Docker Container for Spiders (WIP)
- A docker image for a spider can be generated by editing the Dockerfile to your specific needs, and running `docker build -t [IMAGE_NAME] .`
- In order to run the spider in a container along with Splash, use `docker-compose up` to use the docker-compose.yml file's configuration to launch the spider image along with the required local Splash server. 
- You can retrieve the files generated by running the spider with the `docker cp` command, which copies files from a container to your host machine.
   - In order to do so, find the terminated spider container using `docker ps -a`, and note the first two characters of the container id.
   - Use `docker cp [FIRST_TWO_CHARS]:/usr/src/app .` to get the entire project folder, in which spider logs can be found under `/logs` and .csv output can be found under `/csv`. 

For more detailed usage, please refer to the [Scrapy documentation](https://docs.scrapy.org/en/latest/index.html).

## Troubleshooting
- If your log has messages that say "Connection refused", you most likely do not have the Splash server running on a spider that utlizes SplashRequests. To do so, follow the above instructions in the *Splash* section and install Docker if you haven't already, then use `docker run -p 8050:8050 scrapinghub/splash`.
- If your spider logs have suddenly stopped responding, this most likely is a result of the Splash container reaching memory limits on your machine. The easiest way to solve this in my experience has been to simply restart the container using `docker restart [CONTAINER_ID]`. 
- If you are running the container in a detached state and want to follow the logs in the container, use `docker logs --follow [CONTAINER_ID]`.

## Required Modules and Dependencies
- Python 3.6+
- Scrapy 1.8+
- Scrapy Splash 0.7.2+
- BeautifulSoup4 4.8.2+
- tldextract
