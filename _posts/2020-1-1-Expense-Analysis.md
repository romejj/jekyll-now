---
layout: post
title: Expense Analysis
---

It's the new year! What's more exhilarating than looking back to evaluate how well one has accomplished in his/her financial goals? A lot, actually. In fact, a quarter of millenials in Singapore doesn't keep track of how much they are spending ([Statista](https://www.statista.com/statistics/1101386/singapore-millennial-attitudes-toward-spending/#statisticContainer)). Money is important, so why isn't everyone checking how much money is flowing out of one's pocket? One possible explanation could be that it simply takes a lot of time and effort to do so. Specifically, if one wishes to examine his/her monthly bank statements, it's daunting to manually consolidate lines of transactions and then compare them to a preset budget. Perhaps a better way would be to first copy and paste each line to a separate spreadsheet program, afterwhich numerous aggregations can be easily performed within the program. However, imagine having to repeatedly copy and paste line after line from a single statement like this:
![_config.yml]({{ site.baseurl }}/images/DBS_statement_ex1.png)

This action is then performed for another 11 times, if one wishes to conduct a yearly review of one's expenses. We can now clearly empathise with how painful this process is going to be, and especially so if the person conducts his transactions across multiple credit cards and banks.

This is the very first thought that hit me when I decided to diligently track my expenses, and it's a little embarrassing to admit that I almost called this off. Fortunately, it's also during this period when I also decided to start working on my online portfolio, so this presents the perfect first problem statement for me to put my programming and analytical skills to the test! 

## Outline

Breaking down the problem statement into mini statements to help me with focus on different pipelines:
1. PDF exploration of different bank statements
2. Write a script that automatically extracts and stores transactions from parsed PDF files
3. Write a script that loads all transactions into a csv file
4. Utilise Tableau Public to conduct expense analysis

Caveats:
1. Assuming that most spending done using a card
2. Some amounts have been fabricated as I don’t want people to judge me and my spending habits