# google-news
You can figure out how to scrape Google News using Node.js scraper and Scrapeless Deep SerpApi.

The news.google.com is probably the best place to find news articles! Google News is a collection of real-time, verified news and one of the most trusted and favorite platforms for Internet users.

For the same reason, scraping Google News can be used to collect the latest news articles, trending topics, and related metadata for data analysis, research, or simply reading news. If you want to go a step further, scraping Google News can also help you build your own personalized news application through real-time web scraping.

In this blog post, we will explore how to collect relevant data from Google News using Node.JS and Google News API.

## Why Should We Scrape Google News Results?
Regardless of the industry, scraping data from Google News can greatly facilitate the development of an organization:

Let's take the example of a company in the travel or hospitality industry. Collecting information about travel strategies, safety measures, and tourism trends can help companies predict changes in occupancy rates and plan actions accordingly. They can also use this information to adjust marketing strategies to make them more effective and attract new customers.

On the other hand, investment companies can use financial news to collect data about market developments, regulatory changes, and economic forecasts. The correct use of this data can help them manage risk more effectively and provide more accurate advice to their clients. This can improve their portfolio performance and client satisfaction.

## What data does Google News Scraper collect?

![Google News](https://assets.scrapeless.com/prod/posts/google-news/3479c9876c401bf4ee3d114d827a99e7.png)

**📰 Article Metadata**
- **Headlines**: The title of the news article.
- **Source**: The publisher or news outlet (e.g., BBC, CNN).
- **Publication Date**: When the article was published.
- **Author**: The name of the journalist or contributor.
- **Summary/Snippet**: A brief description or excerpt from the article.
- **URL**: The link to the full article.

**✍️ Content Data**
- **Full Text**: The main body of the news article (requires accessing the source website).
- **Images/Media**: Images, videos, or other media embedded in the article.
- **Keywords/Topics**: Tags or categories associated with the article (e.g., "Politics," "Technology").

**📊 Trending and Popularity Data**
- **Trending Topics**: Current popular topics or stories on Google News.
- **Top Stories**: Articles highlighted as top news for a specific category or region.
- **Search Trends**: Popular search terms related to news topics.

**🧭 Geographical and Demographic Data**
- **Location-Based News**: News articles tailored to specific regions or countries.
- **Language**: The language in which the article is written.

**🧐 Analytics and Insights**
- **Sentiment Analysis**: The tone or sentiment of the article (positive, negative, neutral).
- **Topic Clustering**: Grouping articles by similar themes or subjects.
- **Temporal Analysis**: Tracking how news stories evolve over time.

## Build Your Google News Scraper using Node.js | Step by Step

### Environment preparation
We need to import the following libraries into the Node project.
```JavaScript
const axios = require('axios');
const fs = require('fs');
```

Now, let's create a function to grab some results from Google News:
```JavaScript
async function getNewsData() {
  const headers = {
    "User-Agent":
      "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/101.0.4951.54 Safari/537.36",
  };

  try {
    const response = await axios.get(
      "https://news.google.com/home?hl=en-US&gl=US",
      { headers }
    );
    const html = response.data;
  } catch (error) {
    console.error("Error:", error.message);
  }
}
```

We now set the User-Agent of the Header, which allows us to access Google naturally. Then we use the request library to make a request.

Now, we need to find the required data from the returned data.

![Scrape Google News](https://assets.scrapeless.com/prod/posts/google-news/8eb4034f4be13e474d9468dd3fedb336.png)

If you search through the query, you can find that each result or news exists on the page. Therefore, we need to use regular expressions to match the required data and then process it.

```JavaScript
const regex = /data:(\[.*?\]), sideChannel/s;
const match = html.match(regex);
```

### Scrape Google News data
The specific information in the extracted data group is stored in the form of an array. At this time, you need to compare the specific information on the page and filter out the subscript corresponding to the specific data.
```JavaScript
let resp = [];
const data = JSON.parse(match[1]);
for (const section of data[1][3][1]) {
    if (Array.isArray(section[0])) {
        for (const item of section[0]) {
            const utcTime = new Date(item[4][0] * 1000).toISOString();
            console.log(utcTime)
            resp.push({
                title: item[2],
                source: {
                    name: item[10][2],
                    icon: item[10][22]?.[0] || null,
                    authors: item[item.length - 1]?.[0] || []
                },
                link: item[38],
                thumbnail: item[8]?.[0]?.[13] || null,
                thumbnail_small: item[8]?.[0]?.[0] || null,
                date: utcTime
            });
            break;
        }
    }
}
```

Finally, we have extracted all the data we need.

### Complete code
```JavaScript
const axios = require('axios');
const fs = require('fs');

async function getNewsData() {
    const headers = {
        "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/101.0.4951.54 Safari/537.36"
    };

    try {
        const response = await axios.get("https://news.google.com/home?hl=en-US&gl=US", { headers });
        const html = response.data;

        const regex = /data:(\[.*?\]), sideChannel/s;
        const match = html.match(regex);
        if (!match || !match[0]) {
            throw new Error('No valid JSON data found');
        }
        let resp = [];
        const data = JSON.parse(match[1]);
        for (const section of data[1][3][1]) {
            if (Array.isArray(section[0])) {
                for (const item of section[0]) {
                    const utcTime = new Date(item[4][0] * 1000).toISOString();
                    resp.push({
                        title: item[2],
                        source: {
                            name: item[10][2],
                            icon: item[10][22]?.[0] || null, // Safe access to nested properties
                            authors: item[item.length - 1]?.[0] || []
                        },
                        link: item[38],
                        thumbnail: item[8]?.[0]?.[13] || null,
                        thumbnail_small: item[8]?.[0]?.[0] || null,
                        date: utcTime
                    });
                    break;
                }
            }
        }
        return resp;
    } catch (error) {
        console.error('Error:', error.message);
    }
}

// Execute function
getNewsData().then(data => {
    console.log(data);
});
```

### Crawling results
Now, let's view the results of the code running in the terminal:

![Crawling results](https://assets.scrapeless.com/prod/posts/google-news/fe53bdb42bea6b85e25f0cf51d7398ac.png)

However, in order to save the data in a safer place, we need to do some optimization:
```JavaScript
const fs = require('fs');

fs.writeFileSync('data.json', JSON.stringify(resp, null, 2), 'utf8');
```

The final file content is as follows:

![final results](https://assets.scrapeless.com/prod/posts/google-news/b73344814d1f1adc6eb3f45a12f85035.png)

## Easily scrape Google News with Scrapeless Deep SerpApi

![Deep SerpApi](https://assets.scrapeless.com/prod/posts/google-news/a3e50415cbf2101f26ed5f097252010f.png)

Our Google News API allows you to scrape results from Google News search pages. The API is accessible via the following endpoint: `"engine": "google_news"`. You can use APIDog to complete the data scraping at https://apidocs.scrapeless.com/api-14581677. Alternatively, a quicker way is to watch a live interactive demo directly using Scrapeless Deep SerpApi Playground.

### Why should we use API?
- No need to create a parser from scratch and maintain it.
- Bypass Google's blocking: can automatically solve anti-bot or solve IP blocking.
- No need to pay for proxies and web unlocker additionally.
- No need to use browser automation.

Scrapeless Google News API can easily handle all of the above problems, with a short response time of `~2.33` seconds per request (`~1.47` seconds is amazingly fast). Users only need one API call to get accurate scraped data, which we display using well-structured JSON.

> [**Join our community and get the 500K free usage!**](https://discord.gg/Afc9m9ttWT)

### Using steps
- Step 1. Log in to the [**Scrapeless Playground**](https://app.scrapeless.com/dashboard/products/serpapi?utm_source=official&utm_medium=blog&utm_campaign=google-news).
- Step 2. Find the **Google News** actor and click.
- Step 3. Configure the query parameters.
- Step 4. Click the **Start Search** and get the results.

![Playground Using steps](https://assets.scrapeless.com/prod/posts/google-news/4c709c9a30aa4dd1d9b05e3b4eff049f.png)

## The Bottom Lines
This article discusses two methods to scrape Google News using Node.js. Data collectors who want to have independent scraping tools and want to maintain some flexibility when scraping data can use Node.js as an alternative to interacting with web pages. Follow our steps to easily build your scraper.

In addition, the Google News API is a simple solution that can quickly extract and clean the raw data obtained from the web page and present it in a structured JSON format. Only simple parameter configuration is required to quickly complete data collection.

[**Try it for free now!**](https://app.scrapeless.com/dashboard/products/serpapi?utm_source=official&utm_medium=blog&utm_campaign=google-news)

## FAQs
### Is it legal to scrape Google News?
Yes, it is legal to scrape Google News as it is public information. However, you should be aware of local and regional laws regarding copyright and personal data.

### Does Google remove illegal content?
Yes. Typically, Google will remove or restrict access to the content only in the country/region where it is deemed to be illegal.

### Does Google block web scrapers?
Google's terms and conditions clearly prohibit scraping their services, including search results. Violating these terms may lead to Google services blocking your IP address. As a result, you may have to equip a powerful web unlocker solution.
