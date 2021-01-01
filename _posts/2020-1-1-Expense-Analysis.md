---
layout: post
title: Expense Analysis
---

It's the new year! What's more exhilarating than looking back to evaluate how well one has accomplished in his/her financial goals? A lot, actually. In fact, a quarter of millenials in Singapore doesn't keep track of how much they are spending ([Statista](https://www.statista.com/statistics/1101386/singapore-millennial-attitudes-toward-spending/#statisticContainer)). Money is important, so why isn't everyone checking how much money is flowing out of one's pocket? One possible explanation could be that it simply takes a lot of time and effort to do so. Specifically, if one wishes to examine his/her monthly bank statements, it's daunting to manually consolidate lines of transactions and then compare them to a preset budget. Perhaps a better way would be to first copy and paste each line to a separate spreadsheet program, afterwhich numerous aggregations can be easily performed within the program. However, imagine having to repeatedly copy and paste line after line from a single statement like this:
![_config.yml]({{ site.baseurl }}/images/DBS_statement_ex1.png)

This action is then performed for another 11 times, if one wishes to conduct a yearly review of one's expenses. We can now clearly empathise with how painful this process is going to be, and especially so if the person conducts his/her transactions across multiple credit cards and banks.

This is the very first thought that hit me when I decided to diligently track my expenses, and it's a little embarrassing to admit that I almost called this off. Fortunately, it's also during this period when I also decided to start working on my online portfolio, so this presents the perfect first problem statement for me to put my programming and analytical skills to the test! 

## Outline

We can break down the problem statement into multiple mini problems. Doing so will ensure proper planning and execution of each step, while remaining focused on what we want at the end of this project:  

1. PDF exploration of different bank statements
2. Write a script that automatically extracts and stores transactions from parsed PDF files
3. Write a script that loads all transactions into a csv file
4. Utilise Tableau Public to conduct expense analysis

Caveats:
1. This project assumes that almost all spending was done with credit cards; little or negligible cash transactions were involved.
2. Some amounts have been fabricated as I don’t want people to judge me and my spending habits!

## PDF Exploration

Now, let us take a look at some sample PDF statements. In this project I shall only focus on statements from DBS (The Development Bank of Singapore Limited) and UOB (United Overseas Bank) because well, I only hold accounts in those banks! Fortunately, this can be generalized to other bank statements as the fundamental concepts are essentially the same. 

*First page of DBS:*
![_config.yml]({{ site.baseurl }}/images/DBS_statement_ex2.png)

*Second page of DBS:*
![_config.yml]({{ site.baseurl }}/images/DBS_statement_ex3.png)

*First page of UOB:*
![_config.yml]({{ site.baseurl }}/images/UOB_statement_ex1.png)

*Second page of UOB:*
![_config.yml]({{ site.baseurl }}/images/UOB_statement_ex2.png)

Subsequent pages are irrelevant in this case, as all my transactions are kept within the first two pages of each bank's statement. From examining the statements above we can deduce that transactions are always wrapped within fixed headers and footers:

1. In the case of DBS, the first transaction always starts after "NEW TRANSACTIONS (insert name here)", and the last always comes just before "SUB-TOTAL".
2. In the case of UOB, the first transaction always starts after "PREVIOUS BALANCE", while the last always comes just before "SUB TOTAL". We also need to take note of the footnotes that come in every page of this bank's statements.

Keeping these in mind will tremendously help us in our next data processing stage.

## Transaction Extraction
