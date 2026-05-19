---
layout: post
title: "Introducing `jobscanner`"
date: 2026-05-19 13:00:00 -0000
categories: software
---

This is the first software project I am writing about.
While I am looking for a new job after my PhD, I figured a few software projects on my own would not hurt my chances.
In particular because I would like to get in a software-oriented role.

Job search is annoying and because of the excess of workers on the market (Germany, May 2026), open positions receive a vast number of applications.
I figured that you have an advantage to be an early applicant as recruiters will only have to screen 20 or 30 applications to find ten applicants that are good enough to be shortlisted.
The remaining ones really have to stand out to still have a chance to be considered.
You of course can subscribe to LinkedIn, XING, indeed or other CV graveyards to send you newly posted job listings but if you are like me and have a handful of companies that you really like, then you might not want to rely on those sites that send you the job listings once a day buried under a pile of other job listings that are garbage.

To solve this, I developed a Telegram bot that scrapes websites you like, say every two hours, and notifies you when a new role is posted.
The project is hosted on [Gitlab](https://gitlab.com/ttnick/jobscanner).

During the development I overhauled the product twice but now I am sort of happy with the architecture it has.
There is still work in progress, so much upfront, but a basic functional version exists and it works quite well for me.

My first impression was that Python with some `BeautifulSoup` would already do the trick but this is me just continuing to be oblivious to Dark Magic JavaScript frameworks that dynamicall extend the DOM based on human interaction with it.
This is how I ended up choosing `Playwright` to do the heavy lifting.

The idea is as follows, the user can provide a TOML file with some information about the job listing websites of the companies he or she likes.
This is, in a nutshell, the URL, a regular expression that matches links (i.e. `<a>` tags) that correspond to job listings.
You then also provide a way to extract the job title by providing a path from the `<a>` to this title.
One might think that it is just the inner text of the `<a>` but this is (most of the time even) not the case; often the `<a>` wraps around a bigger container with more information than the job title.
To find the correct location of the job title, I developed a very simple DSL to reach from the `<a>` tag to the title.

There are two more things that one might have to take care of
 1. Pagination: Some websites might distribute job listings to multiple pages and do not show you all of them at once.
 2. Filters: Often filters are just applied with a GET request; in that case you can just use the resulting URL. However, if this is not the case (say, it was a POST request instead), then you need to let Playwright fill these forms for you. Similarly, accepting cookies is something you might have to do first before even properly accessing the site.

Both problems are solved in a similar way: You just provide actions on some CSS selector (clicking, filling, checking, selecting, ...).
There is another DSL to give `Playwright` the right commands.
After that the scraping is straight-forward: 
 1. Go to the website.
 2. Perform pre-scrape tasks (as accepting cookis, fill out forms, ...).
 3. Find links that match the provided regex.
 4. Perform post-scrape tasks (it's possible, but I did not have a need for it yet).
 5. Compare all scraped jobs against those that you already know (they are stored in an SQLite database) and also store your new jobs.
 6. Broadcast new jobs via Telegram bot.

The latest architecture separates certain steps:
 * `scraper` a class that implements everything necessary to scrape a website (including pre and post scrape tasks). The one I implemented is essentially calling Playwright functions
 * `storage` a class that takes care of registering new jobs and identifying those you already know. Also stores the users of the Telegram bot. This is done here in SQLite.
 * `interface` a class that handles the broadcast of new jobs but it also can receive commands (e.g., to initiate a manual scrape). This calls the awesome `python-telegram-bot` wrapper for the Telegram API.
 * `main` essentially just orchestrates the previous three modules to work with each other.

The nice thing about modulizing the three core parts is that it also becomes now easy to exchange them (SQLite against MongoDB, Playwright implementation against Selenium, Telegram against E-Mail).

To run your own `jobscanner` bot you only have to follow a few steps (see also [Documentation](https://gitlab.com/ttnick/jobscanner/-/blob/main/DOCUMENTATION.md)):
 1. Install `asyncio`, `playwright`, `dotenv`, `toml`, and `python-telegram-bot`.
 2. Start chromium with remote debugging: 
   ```
   chromium --remote-debugging-port=9222
   ```
   This is necessary at the moment to evade Cloudflare's bot detection, which i have not figured out completely yet in headless mode.
 3. Get a Telegram access token from [@BotFather](https://t.me/botfather) and provide this together with some other data (CDP url, scrape interval, ...) in an `.env`.
 4. Define which company websites to scrape for job listings.

The last part is the only really somewhat part as it requires you to inspect some HTML elements yourself.
I would love to use some AI for that in the future but I am not there yet.

Here are some examples that I think are instructive from my search.
Since I am located in Dortmund, Germany, the companies described are also from that city.
(As of today, neither of them hired me.)

### Borussia Dortmund
This is not where you can apply to become a professional football player.
However, there are a ton of behind-the-scenes jobs at major football clubs that one can look into.
You can find their job listings at [`https://karriere.bvb.de/#jobs`](https://karriere.bvb.de/#jobs).
This is also the only mandatory field you have to provide (`url`).
Given that `jobscanner` can perform certian pre scrape tasks, it is tempting to now apply filters w.r.t. job category or career levels. 
However, in case of this website, this is not really necessary since these filters are apparently handled using something similar as a GET request.
If we filter for say `IT`, the URL changes to [`https://karriere.bvb.de/#jobs:category=["IT"]`](https://karriere.bvb.de/#jobs:category=%5B%22IT%22%5D).
So by just using this URL instead, you can omit the whole pre scrape task thing.
However, this page has not only links to job listings but also to their podcast or their kununu page.
How do we get only those links that are interesting fo us?
The links to the job listings have a very simple structure; they all look something like `https://karriere.bvb.de/jobs/123456/IT-Admin/`.
`jobscanner` allows you to provide a regular expression on the URLs that are releveant for you.
In this particular example it is in fact enough to look for the infix `/jobs/` to get the right collection of links, so this is what you write in your `url_pattern`.
No regular expression needed.
Also the `title_path` is straight forward: The `<a>` tag wraps around three `div` tags of which one you need the first that contains the job title.
So in total, you get:
```
[companies.BVB]
url = "https://karriere.bvb.de/#jobs:category=%5B%22IT%22%5D"
url_pattern = "/jobs/"
title_path = "this|find:div|text"
```

### Dortmund Airport
Aviation fascinates me a lot and they have interesting optimization problems to solve.
Their job listings can be found at [`https://www.dortmund-airport.com/de/unternehmen/stellenangebote`](https://www.dortmund-airport.com/de/unternehmen/stellenangebote). 
They do not have any filters so we can only scrape for all jobs.
This is fine for me here as there are not too many jobs anyway.
The job links all have `/stellenangebote/` as infix but the main page itself also has this one (without the trailing `/`).
To be on the safe side, let us just require that we have also something behind the `/`.
Here we use a regular expression `/stellenangebote/.+`.
The title is just the inner text of the link, so this is as easy as it gets:
```
[companies.DTM]
url = "https://www.dortmund-airport.com/de/unternehmen/stellenangebote"
url_pattern = "/stellenangebote/.+"
title_path = "this|text"
```

### Adesso
One of the biggest software companies in Germany; there are tons of open positions, so we want to be sure to apply filters here.
The jobs can be found at [`https://jobs.adesso-group.com`](https://jobs.adesso-group.com).
At first glance, this website seems hard to scrape because they use big boxes with a bunch of other stuff inside the `<a>` tag but more crucially: They do not show you all the jobs at once.
Instead there is a button at the bottom that loads 15 more jobs with every time you click.
Luckily, there is a way around that, so you still do not need pre scrape tasks.
A good idea is it to apply your filters as you want them and change the view to *list*, this gives you a much leaner container.
Among all the GET parameters that you find in the URL, one is `pageSize` which is just the number of jobs that are displayed at once.
By just setting this value to a sufficiently high number (say 1000), you actually get to see all jobs on one page where your filters apply (which are also in the URL).
So the `url` you have to provide could for example be: [`https://jobs.adesso-group.com/?resultsView=list&region=dortmund&currentPage=1&pageSize=1000&orderBy=datePosted&isDesc=true`](https://jobs.adesso-group.com/?resultsView=list&region=dortmund&currentPage=1&pageSize=1000&orderBy=datePosted&isDesc=true).
Moreover, with the list view the `title_path` is straight forward by moving into the `<div>` of class `content` and then in the `<div>` with class `title`.
Overall you get something like this:
```
[companies.adesso]
url = "https://jobs.adesso-group.com/?resultsView=list&region=dortmund&currentPage=1&pageSize=1000&orderBy=datePosted&isDesc=true"
url_pattern = "/job-invite/"
title_path = "this|find:.content|find:.title|text"
```

### Materna
Materna is *the other* big software company in Dortmund and here, we have to put a few more things together to make `jobscanner` work.
On one hand, Materna detects bots using [Cloudflare](https://en.wikipedia.org/wiki/Cloudflare), which does not work well with plain Playwright, the underlying browser automation library behind `jobscanner`.
There are some ways to circumvent this but I am still looking into the details for that.
Cloudflare is essentially the reason why you need to connect a (proper, human) chromium browser via CDP to `jobscanner`.
Another thing that Materna has that we have not seen in the other examples are pagination links.
There is no way to display all the jobs all at once, they will distribute among multiple pages, each of which will reload the page.
Of course on could just provide a separate instance in you `companies.toml` to all the pages but this does not work either for two reasons: 
 1. You don't know how any pages (sure you can find a trivial upper bound maybe).
 2. The filters you probably want to apply are not sent via GET but via POST, so you always need to send them manually and cannot just request a specific URL. This is just not very handy to solve just using copies in you TOML.

So let's solve this: The job listings can be searched at [`https://karriere.materna.de/stellenmarkt/stellenangebote.html`](https://karriere.materna.de/stellenmarkt/stellenangebote.html).
Let us say for the moment, you have filters applied already, then we get the easy stuff out of the way.
You simply have to filter links for the pattern `/stellenmarkt/.*\.html`.
This pattern will also give you the main page above (but only once in your first scrape, I think we can live with that); it omits the pagination links and all other irrelevant links though.
The title is indeed just the inner text of the `<a>` tag.
Great but how do we get to this point? In contrast to the other sites, Materna forces you to make a selection regarding cookies (citing DHH: Cookie banners are the monuments of why Europe is losing in tech), so we have to click this one away with `click:.sg-cookie-optin-box-button-accept-essential`.
Then to get actually access to all the filters, you have to expand the form using this little arrow under the visible form fields (`click:.icon_arrow`).
This is by the way not optional: One could entertain the idea that you can just look for the HTMl elements and make some actions on them but without certain clicks that trigger some JavaScript, these elements might be disabled or not even generated in the DOM yet.
Here it is really important for `jobscanner` to act like a human.
Then we filter, for example for the location (`click:[title='Auswahl Standort']`) and check the relevant box (`check:#ui-multiselect-countr-option-3`; this is Dortmund).  
Finally, recall that we have pagination links, and the little *page forward* link is at the bottom of the page `>`.
Since a single symbol is quite weak as identifier, we can also use other CSS selectors; here I used the `title` attribute of the `<a>` tag (`[title='Nächste Seite']`).
All in all, this yields:
```
[companies.materna]
url = "https://karriere.materna.de/stellenmarkt/stellenangebote.html"
url_pattern = '/stellenmarkt/.*\.html$'
title_path = "this|text"
pagination = "[title='Nächste Seite']"
pre_scrape_tasks = "click:.sg-cookie-optin-box-button-accept-essential|click:.icon_arrow|click:[title='Auswahl Standort']|check:#ui-multiselect-countr-option-3"
```
