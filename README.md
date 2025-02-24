# Parsing HTML With jsoup

[![Promo](https://github.com/luminati-io/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://brightdata.com/) 

This guide explains how to parse HTML with `jsoup` in Java. You will learn how to use DOM methods, handle pagination, and optimize your parsing workflow.

- [Using DOM Methods With Jsoup](#using-dom-methods-with-jsoup)
  - [getElementById](#getelementbyid)
  - [getElementsByTag](#getelementsbytag)
  - [getElementsByClass](#getelementsbyclass)
  - [getElementsByAttribute](#getelementsbyattribute)
- [Advanced Techniques](#advanced-techniques)
  - [CSS Selectors](#css-selectors)
  - [Handling Pagination](#handling-pagination)
- [Putting Everything Together](#putting-everything-together)

## Getting Started

This tutorial assumes using [Maven](https://maven.apache.org/) for dependency management.

Once you’ve got Maven installed, create a new Java project called `jsoup-scraper`:

```bash
mvn archetype:generate -DgroupId=com.example -DartifactId=jsoup-scraper -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
```

To add relevant dependencies, replace the code in `pom.xml` with the code below:

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.example</groupId>
  <artifactId>jsoup-scraper</artifactId>
  <packaging>jar</packaging>
  <version>1.0-SNAPSHOT</version>
  <name>jsoup-scraper</name>
  <url>http://maven.apache.org</url>
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.jsoup</groupId>
        <artifactId>jsoup</artifactId>
        <version>1.16.1</version>
    </dependency>
  </dependencies>
  <properties>
    <maven.compiler.source>17</maven.compiler.source>
    <maven.compiler.target>17</maven.compiler.target>
</properties>
</project>
```

Now paste the below code into `App.java`:

```java
package com.example;

import org.jsoup.Jsoup;
import org.jsoup.nodes.Document;
import org.jsoup.nodes.Element;
import org.jsoup.select.Elements;

public class App {
    public static void main(String[] args) {

        String url = "https://books.toscrape.com";
        int pageCount = 1;

        while (pageCount <= 1) {

            try {
                System.out.println("---------------------PAGE "+pageCount+"--------------------------");

                //connect to a website and get its HTML
                Document doc = Jsoup.connect(url).get();
            
                //print the title
                System.out.println("Page Title: " + doc.title());
            
                
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
        System.out.println("Total pages scraped: "+(pageCount-1));
    }
}
```

- `Jsoup.connect("https://books.toscrape.com").get()`: This line fetches the page and returns a `Document` object that you can manipulate.
- `doc.title()` returns the title in the HTML document, in this case: `All products | Books to Scrape - Sandbox`.

## Using DOM Methods With Jsoup

`jsoup` contains a variety of methods for finding elements in the DOM(Document Object Model). We can use any of the following to find page elements easily.

- `getElementById()`: Find an element using its `id`.
- `getElementsByClass()`: Find all elements using their CSS class.
- `getElementsByTag()`: Find all elements using their HTML tag.
- `getElementsByAttribute()`: Find all elements containing a certain attribute.

### getElementById

On the website we are scraping, the sidebar contains a `div` with an `id` of `promotions_left`:

![Inspect the sidebar](https://brightdata.com/wp-content/uploads/2025/02/Inspect-the-sidebar.png)

```java
//get by Id
Element sidebar = doc.getElementById("promotions_left");

System.out.println("Sidebar: " + sidebar);
```

This code outputs the HTML element you see in the Inspect page.

```
Sidebar: <div id="promotions_left">
</div>
```

### getElementsByTag

`getElementsByTag()` allows to find all elements on the page with a certain tag. On this page, where each book is contained in a unique `article` tag:

![Inspect books](https://brightdata.com/wp-content/uploads/2025/02/Inspect-books.png)

The code below returns an array of books that will provide the foundation for the rest of the data.

```java
//get by tag
Elements books = doc.getElementsByTag("article");
```

### getElementsByClass

Let's inspect the price of a book. The class is `price_color`:

![Inspect price](https://brightdata.com/wp-content/uploads/2025/02/Inspect-price.png)

The below code snippet finds all elements of the `price_color` class and prints the text of the first one using `.first().text()`:

```java
System.out.println("Price: " + book.getElementsByClass("price_color").first().text());
```

### getElementsByAttribute

Let's use `getElementsByAttribute("href")` to find all elements with an `href` attribute:

```java
//get by attribute
Elements hrefs = book.getElementsByAttribute("href");
System.out.println("Link: https://books.toscrape.com/" + hrefs.first().attr("href"));
```

## Advanced Techniques

### CSS Selectors

To find elements by multiple criteria, let's pass CSS selectors to the `select()` method. This will return an array of all objects matching the selector. In the next code snippet, we use `li[class='next']` to find all `li` items with the `next` class:

```java
Elements nextPage = doc.select("li[class='next']");
```

### Handling Pagination

To handle pagination, we start by using `nextPage.first()` to obtain the first element from the array. We then call `getElementsByAttribute("href").attr("href")` on that element to extract its `href` value.

Since after page 2, the word `catalogue` is removed from the links,  we add `href` back if does not contain `catalogue`. After that, we combine this updated link with our base URL to obtain the URL for the next page.

```java
if (!nextPage.isEmpty()) {
    String nextUrl = nextPage.first().getElementsByAttribute("href").attr("href");
    if (!nextUrl.contains("catalogue")) {
        nextUrl = "catalogue/"+nextUrl;
    } 
    url = "https://books.toscrape.com/" + nextUrl;
    pageCount++;
}
```

## Putting Everything Together

Here is the final Java code. To scrape more than one page, simply change the `1` in `while (pageCount <= 1)`. E.g., if you want to scrape 4 pages, use `while (pageCount <= 4)`.

```java
import org.jsoup.Jsoup;
import org.jsoup.nodes.Document;
import org.jsoup.nodes.Element;
import org.jsoup.select.Elements;

public class App {
    public static void main(String[] args) {

        String url = "https://books.toscrape.com";
        int pageCount = 1;

        while (pageCount <= 1) {

            try {
                System.out.println("---------------------PAGE "+pageCount+"--------------------------");

                //connect to a website and get its HTML
                Document doc = Jsoup.connect(url).get();
            
                //print the title
                System.out.println("Page Title: " + doc.title());
            
                //get by Id
                Element sidebar = doc.getElementById("promotions_left");

                System.out.println("Sidebar: " + sidebar);

                //get by tag
                Elements books = doc.getElementsByTag("article");

                for (Element book : books) {
                    System.out.println("------Book------");
                    System.out.println("Title: " + book.getElementsByTag("img").first().attr("alt"));
                    System.out.println("Price: " + book.getElementsByClass("price_color").first().text());
                    System.out.println("Availability: " + book.getElementsByClass("instock availability").first().text());

                    //get by attribute
                    Elements hrefs = book.getElementsByAttribute("href");
                    System.out.println("Link: https://books.toscrape.com/" + hrefs.first().attr("href"));
                }

                //find the next button using its CSS selector
                Elements nextPage = doc.select("li[class='next']");
                if (!nextPage.isEmpty()) {
                    String nextUrl = nextPage.first().getElementsByAttribute("href").attr("href");
                    if (!nextUrl.contains("catalogue")) {
                        nextUrl = "catalogue/"+nextUrl;
                    } 
                    url = "https://books.toscrape.com/" + nextUrl;
                    pageCount++;
                }

            } catch (Exception e) {
                e.printStackTrace();
            }
        }
        System.out.println("Total pages scraped: "+(pageCount-1));
    }
}
```

Compile the code:

```bash
mvn package
```

Now you can run it:

```bash
mvn exec:java -Dexec.mainClass="com.example.App"
```

Here is the output from the first page.

```
---------------------PAGE 1--------------------------
Page Title: All products | Books to Scrape - Sandbox
Sidebar: <div id="promotions_left">
</div>
------Book------
Title: A Light in the Attic
Price: £51.77
Availability: In stock
Link: https://books.toscrape.com/catalogue/a-light-in-the-attic_1000/index.html
------Book------
Title: Tipping the Velvet
Price: £53.74
Availability: In stock
Link: https://books.toscrape.com/catalogue/tipping-the-velvet_999/index.html
------Book------
Title: Soumission
Price: £50.10
Availability: In stock
Link: https://books.toscrape.com/catalogue/soumission_998/index.html
------Book------
Title: Sharp Objects
Price: £47.82
Availability: In stock
Link: https://books.toscrape.com/catalogue/sharp-objects_997/index.html
------Book------
Title: Sapiens: A Brief History of Humankind
Price: £54.23
Availability: In stock
Link: https://books.toscrape.com/catalogue/sapiens-a-brief-history-of-humankind_996/index.html
------Book------
Title: The Requiem Red
Price: £22.65
Availability: In stock
Link: https://books.toscrape.com/catalogue/the-requiem-red_995/index.html
------Book------
Title: The Dirty Little Secrets of Getting Your Dream Job
Price: £33.34
Availability: In stock
Link: https://books.toscrape.com/catalogue/the-dirty-little-secrets-of-getting-your-dream-job_994/index.html
------Book------
Title: The Coming Woman: A Novel Based on the Life of the Infamous Feminist, Victoria Woodhull
Price: £17.93
Availability: In stock
Link: https://books.toscrape.com/catalogue/the-coming-woman-a-novel-based-on-the-life-of-the-infamous-feminist-victoria-woodhull_993/index.html
------Book------
Title: The Boys in the Boat: Nine Americans and Their Epic Quest for Gold at the 1936 Berlin Olympics
Price: £22.60
Availability: In stock
Link: https://books.toscrape.com/catalogue/the-boys-in-the-boat-nine-americans-and-their-epic-quest-for-gold-at-the-1936-berlin-olympics_992/index.html
------Book------
Title: The Black Maria
Price: £52.15
Availability: In stock
Link: https://books.toscrape.com/catalogue/the-black-maria_991/index.html
------Book------
Title: Starving Hearts (Triangular Trade Trilogy, #1)
Price: £13.99
Availability: In stock
Link: https://books.toscrape.com/catalogue/starving-hearts-triangular-trade-trilogy-1_990/index.html
------Book------
Title: Shakespeare's Sonnets
Price: £20.66
Availability: In stock
Link: https://books.toscrape.com/catalogue/shakespeares-sonnets_989/index.html
------Book------
Title: Set Me Free
Price: £17.46
Availability: In stock
Link: https://books.toscrape.com/catalogue/set-me-free_988/index.html
------Book------
Title: Scott Pilgrim's Precious Little Life (Scott Pilgrim #1)
Price: £52.29
Availability: In stock
Link: https://books.toscrape.com/catalogue/scott-pilgrims-precious-little-life-scott-pilgrim-1_987/index.html
------Book------
Title: Rip it Up and Start Again
Price: £35.02
Availability: In stock
Link: https://books.toscrape.com/catalogue/rip-it-up-and-start-again_986/index.html
------Book------
Title: Our Band Could Be Your Life: Scenes from the American Indie Underground, 1981-1991
Price: £57.25
Availability: In stock
Link: https://books.toscrape.com/catalogue/our-band-could-be-your-life-scenes-from-the-american-indie-underground-1981-1991_985/index.html
------Book------
Title: Olio
Price: £23.88
Availability: In stock
Link: https://books.toscrape.com/catalogue/olio_984/index.html
------Book------
Title: Mesaerion: The Best Science Fiction Stories 1800-1849
Price: £37.59
Availability: In stock
Link: https://books.toscrape.com/catalogue/mesaerion-the-best-science-fiction-stories-1800-1849_983/index.html
------Book------
Title: Libertarianism for Beginners
Price: £51.33
Availability: In stock
Link: https://books.toscrape.com/catalogue/libertarianism-for-beginners_982/index.html
------Book------
Title: It's Only the Himalayas
Price: £45.17
Availability: In stock
Link: https://books.toscrape.com/catalogue/its-only-the-himalayas_981/index.html
Total pages scraped: 1
```

## Conclusion

Scraping dynamic sites like product listings, news, or research data can be challenging. [Bright Data’s tools](https://brightdata.com/products) help you scale your efforts:

- **Residential Proxies:** Bypass IP bans and geo-restrictions.
- **Scraping Browser:** Easily handle JavaScript-heavy sites.
- **Ready-to-Use Datasets:** Get structured data without scraping.

Combine these with jsoup for efficient, low-risk data extraction. Try them for free today!