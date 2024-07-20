# App Icons Dataset

This repository contains a dataset of app icons fetched from [macosicongallery](https://macosicongallery.com). The dataset was generated using Python and Scrapy for web scraping, and LLaVA for generating captions locally on a MacBook. Please note that some captions may have formatting issues.

## Dataset

The dataset includes:
- High-quality images of macOS application icons.
- Captions for each icon generated using LLaVA.

## Setup

1. Clone the repository:
    ```bash
    git clone https://github.com/AurelDeveloper/app-icons-dataset.git
    cd app-icons-dataset
    ```

## Python 🐍 Scraper

    ```python
    import scrapy

    class macOSIconGallerySpider(scrapy.Spider):
        name = 'macos_icon_gallery_spider'
        allowed_domains = ['macosicongallery.com']
        start_urls = ['https://www.macosicongallery.com/']

    custom_settings = {
        'FEED_FORMAT': 'json',
        'FEED_URI': 'macos_icon_gallery_build.json'
    }

    def start_requests(self):
        yield scrapy.Request(url=self.start_urls[0], callback=self.parse, meta={'page': 1})

    def parse(self, response):
        # Überprüfung auf 404-Fehlerseite
        if response.status == 404:
            self.logger.info('Reached a page with 404 error, stopping the spider.')
            return

        links = response.css('a::attr(href)').getall()
        filtered_links = [link for link in links if link.startswith('/icons/')]
        for link in filtered_links:
            yield response.follow(link, self.parse_icon_page)

        # Bereitet den nächsten Seitenaufruf vor
        next_page = response.meta.get('page') + 1
        next_page_url = f'https://www.macosicongallery.com/p/{next_page}'
        yield scrapy.Request(next_page_url, callback=self.parse, meta={'page': next_page})

    def parse_icon_page(self, response):
        icon_info = {
            'app_name': response.xpath('//dt[text()="App"]/following-sibling::dd[1]/text()').get().strip(),
            'category': response.xpath('//dt[text()="Category"]/following-sibling::dd[1]/text()').get().strip(),
            'image_url': response.css('span.icon-wrapper img::attr(src)').get()
        }
        yield icon_info
    ```
