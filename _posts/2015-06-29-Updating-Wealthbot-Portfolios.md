---
layout: post
title:  "Updating Wealthbot Portfolios"
date: 2015-06-29 19:29:56
categories: opensource symfony
comments: true
---
I've been getting a lot of questions lately about the portfolio models pre-installed in the [wealthbot.io demo](http://demo.wealthbot.io). 

> Should I use these models? 

![webo models](/images/webo_org_models.png)

> What if I want to recommend my own portfolios models?

My answer? That's fantastic! 

In fact, I want every Investment Advisor that installs Webo or any indiviual that sets up Webo up for home use to define his or her own portfolio models. That's where Webo shines, is in customizability and flexiblity. 

And, I want to clarify that the asset allocation models in the demo version of Webo are **not investment advice**. The data is just to illustrate what the system looks like when it's set up.

But the questions did get me thinking... 

Should we preset some protfolios? Not ones that we personally created and recommend, but some that are freely available online and put together by people a lot smarter than me. 

Seems like a good idea.

Plus it gives me an interesting way of exploring how **Symfony's Fixtures** are used in Webo.

## Symfony Fixtures
Data is prepopulated in all wealthbot.io installs using [Symfony's Doctrorine Fixtures](symfony.com/doc/current/bundles/DoctrineFixturesBundle/index.html). As the documentation states: 

> Fixtures are used to load a controlled set of data into a database. This data can be used for testing or could be the initial data required for the application to run smoothly.

That being said, we are using Fixtures in a bit of an unusual way for wealthbot.io. We're loading both seed data and unit test data with the same Fixtures files. BIG no-no, but the plan is to seperate that out in the next few releases.

---

In this exercise, I'll be updating our fixtures to include the most basic portfolio from [Boglehead's Lazy Portfolios](www.bogleheads.org/wiki/Lazy_portfolios). Rick Ferri's Two Fund Portfolio.

## Two Fund Portfolio

Rick Ferri's portfolio is as simple as simple can be. Two index funds, 60% stocks, 40% bonds. 

% Allocation | Fund | ETF Fund 
--- | --- | ---
40% | Vanguard Total Bond Market Index Fund | BND (.07%)
60% | Vanguard Total World Stock Index Fund	| VT (.05%)


## Updating Webo Fixtures
Something to keep in mind: I'm not the technical expert on this project, so hopefully the way I tackled this fix will help other less technically-experienced folks like myself get comfortable working with Webo. 

Also, this writeup assumes you've followed our [super-simple installation instructions](https://github.com/wealthbot-io/wealthbot) and all is well at [local.wealthbot.io](http://local.wealthbot.io).

First things first, let's open up the terminal and create a new branch:

```
git checkout -b portfolios
```

OK, now we can change some stuff and no biggie if we mess things up. I can't remember where exactly the file with model date is. But, I know the name of an existing model!

![webo models](/images/webo_models.png)

Just copy and paste into the Github search field and all will be revealed. 

![search github for Webo 100% Stock](/images/webo_search_results.png)

OK, now let's open this file and some others in our favorite text editor. Actually, I'm going to open the whole folder:

![Sublime LoadCecRiaData](/images/LoadCecRiaData.png)

As you can see from the array for 'Webo 100% Stocks' here are the requirements for a new portfolio model:

* Asset Class
* Subclass
* Security symbol
* Percent of portfolio


Preloaded in the *$categories* array we have 4 asset classes and a few subclasses under each asset class. I present them here with their associated index numbers. 

| Index | **Asset Class** | Index - Subclass |
| ----- | --------------------------- | -------------------- |
| 0 | *Domestic Stocks* | 0 - Large, 1 - Large Value, 2 - Small, 3 - Small Value |
| 1 | *International Stocks* | 0 - Large, 1 - Large Value, 2 - Small, 3 - Small Value, 4 - Emerging Markets, 5 - REITS |
| 2 | *Alternatives* | 0 - Commodities, 1 - REITs, 2- International REITs |
| 3 | *Bonds* | 0 - Intermedieate, 1 - Short, 2- Long |


These asset classes aren't the best setup for our two-fund, world stock market portfolio. But that's a problem for another post.

We'll make do. Here is the new **Rick Ferri Portfolio** Fixture I wrote into the *$strategy* array. 

```PHP
array(
                'name' => 'Rick Ferri Two Fund Portfolio',
                'index' => 'rf_two_fund_portfolio',
                'risk_rating' => 3,
                'is_assumption_locked' => 0,
                'entities' => array(
                    array('asset_class_index' => 2, 'subclass_index' => 0, 'security' => 'BND', 'muni_substitution_security' => null, 'tax_loss_harvesting_security' => null, 'percent' => 40),  
                    array('asset_class_index' => 0, 'subclass_index' => 0, 'security' => 'VT', 'muni_substitution_security' => null, 'tax_loss_harvesting_security' => null, 'percent' => 60),   
                )
            ),
```

Let's quickly cover the important settings in this array. 

* *risk_rating*: this rating recommends your model based on the client's risk assessment number
* *asset_class_index* & *subclass_index*: see table above
* *security* = the security symbol. 
* *muni_substitution_security* & *tax_loss_harvesting_security* = which security to use in the MI and TLH senarios defined for this portfolio

Securities Fixtures are in LoadSecurityData.php, and I quickly checked for 'BND' and 'VT'. 'BND' already existed, but 'VT' did not. Here's the array I added just below 'Vanguard Total Stock Market ETF', in '$securites'.

```PHP
array(
            'name' => 'Vanguard Total World Stock ETF',
            'symbol' => 'VT',
            'security_type' => 'EQ',
            'exp_ratio' => 0.17
        ),
``` 

## Loading Fixtures
Almost done now. We just have to reload the fixtures. Be forewarned, this will purge the wealthbot database.

Hopefully your local install is running by now. Go to your wealthbot folder and use `vagrant ssh` to connect with the server. Now:

```
cd /srv/wealthbot
```

```
php app/console doctrine:fixtures:load
```

Sit back and watch the magic.

![Fixtures Load](/images/load_fixtures.png)

Let's go to [http://local.wealthbot.io/](http://local.wealthbot.io/) and make sure our new portfolio's loaded correctly.

![New Portfolio](/images/new_portfolio.png)

Lookin' good Webo. Looking real good.

Don't forget to push your branch to master and to [send us a pull request](https://github.com/wealthbot-io/wealthbot/blob/master/CONTRIBUTING.md)! 


 
