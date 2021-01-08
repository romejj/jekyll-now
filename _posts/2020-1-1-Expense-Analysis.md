---
layout: post
title: Expense Analysis
---

It's the new year! What's more exhilarating than looking back to evaluate how well one has accomplished in his/her financial goals? A lot, actually. In fact, a quarter of millenials in Singapore doesn't keep track of how much they are spending ([Statista](https://www.statista.com/statistics/1101386/singapore-millennial-attitudes-toward-spending/#statisticContainer)). Money is important, so why isn't everyone checking how much money is flowing out of one's pocket? One possible explanation could be that it simply takes a lot of time and effort to do so. Specifically, if one wishes to examine his/her monthly bank statements, it's daunting to manually consolidate lines of transactions and then compare them to a preset budget. Perhaps a better way would be to first copy and paste each line to a separate spreadsheet program, afterwhich numerous aggregations can be easily performed within the program. However, imagine having to repeatedly copy and paste line after line from a single statement like this:
![_config.yml]({{ site.baseurl }}/images/DBS_statement_ex1.png)

This action is then performed for another 11 times, if one wishes to conduct a yearly review of one's expenses. We can now clearly empathise with how painful this process is going to be, and especially so if the person conducts his/her transactions across multiple credit cards and banks.

This is the very first thought that hit me when I decided to diligently track my expenses, and it's a little embarrassing to admit that I almost called this off. Fortunately, it's also during this period when I also decided to start working on my online portfolio, so this presents the perfect first problem statement for me to put my programming and analytical skills to the test! 

## Outline

We can break down the problem statement into multiple mini problems. Doing so will ensure proper planning and execution of each step, while remaining focused on what we want to achieve at the end of this project:  

1. PDF exploration of different bank statements
2. Write a script that automatically extracts and stores transactions from parsed PDF files
3. Write a script that loads all transactions into a csv file
4. Utilise Tableau Public to conduct expense analysis

Caveats:
1. This project assumes that almost all spending was done with credit cards; little or negligible cash transactions were involved.
2. Most amounts have been fabricated as I don’t want people to judge me and my spending habits!

## PDF Exploration

Now, let us take a look at some sample PDF statements. In this project I shall only focus on statements from DBS (The Development Bank of Singapore Limited) and UOB (United Overseas Bank) because well, I only hold accounts in those banks! Fortunately, this can be generalized to other bank statements as the fundamental concepts are essentially the same. 

***First page of DBS:***
![_config.yml]({{ site.baseurl }}/images/DBS_statement_ex2.png)

***Second page of DBS:***
![_config.yml]({{ site.baseurl }}/images/DBS_statement_ex3.png)

***First page of UOB:***
![_config.yml]({{ site.baseurl }}/images/UOB_statement_ex1.png)

***Second page of UOB:***
![_config.yml]({{ site.baseurl }}/images/UOB_statement_ex2.png)

Subsequent pages are irrelevant in this case, as all my transactions are kept within the first two pages of each bank's statement. From examining the statements above we can deduce that transactions are always wrapped between fixed headers and footers:

*Scenario 1: All transactions are contained within first page:*

| **Bank**            | **DBS**                               | **UOB**            |
|---------------------|---------------------------------------|--------------------|
| **1st page header** | "NEW TRANSACTIONS (insert name here)" | "PREVIOUS BALANCE" |
| **1st page footer** | "SUB-TOTAL"                           | "SUB TOTAL"        |
| **2nd page header** | N/A                                   | N/A                |
| **2nd page footer** | N/A                                   | N/A                |

*Scenario 2: Transactions are listed in both first page and second page:*

| **Bank**            | **DBS**                               | **UOB**            |
|---------------------|---------------------------------------|--------------------|
| **1st page header** | "NEW TRANSACTIONS (insert name here)" | "PREVIOUS BALANCE" |
| **1st page footer** | No footer                             | "Pleasenote"       |
| **2nd page header** | "2 of 3"                              | "Date Date SGD"    |
| **2nd page footer** | "SUB-TOTAL"                           | "SUB TOTAL"        |

Keeping these in mind will tremendously help us in our next data processing stage.

## Transaction Extraction
We need to find a suitable library that is able to read PDF pages and store them as objects. *pdfplumber* (well documented [here](https://github.com/jsvine/pdfplumber)) serves this purpose well; it not only stores them as objects but also provides a range of useful operations that can be executed on these objects to serve this project's objectives.

Remember when I mentioned in the previous section about taking note of the fixed headers and footers? It comes in handy here; when I parsed the first page of the sample DBS statement and then partitioned based on the fixed header,

{% highlight ruby %}
with pdfplumber.open(dbs_source_dir / dbs_pdf_file) as pdf:
    first_page = pdf.pages[0]
    first_page_text = first_page.extract_text()
    first_page_txns_raw = first_page_text.partition("NEW TRANSACTIONS JEROME KO JIA JIN")[2]
    print(first_page_txns_raw)
{% endhighlight %}

I obtained the following:

![_config.yml]({{ site.baseurl }}/images/parsed_DBS_fp.png)

So far so good! With just a few blocks of code we have managed to store transactions in a string. However, we want to accomplish more here. As we can see the transactions are still not very clean, and we should also compartmentalize each transaction according to dates, transaction description, and transactional amount so we can then easily write them into a spreadsheet later. Storing the transactions as a list will help accomplish this. We first examine whitespace characters within the string to see how we can split the string accordingly:

{% highlight ruby %}
print(repr(first_page_txns_raw))
{% endhighlight %}

![_config.yml]({{ site.baseurl }}/images/print(repr(first_page_txns_raw)).png)

Nice! It looks like the transactions can be neatly split by the newline character, returning a list with each element representing a single transaction. Each element can then be further separated to dates, description and amount by spaces. The result is a list within a list.

{% highlight ruby %}
def filter_legitimate_txns(txns):
    txns_split = txns.split("\n")
    txns_double_split = [txn.split() for txn in txns_split]

pprint.pprint(filter_legitimate_txns(first_page_txns_raw))
{% endhighlight %}

![_config.yml]({{ site.baseurl }}/images/list_split.png)

The last step involves cleaning up the transactions in the list. Since each transaction is defined by its dates, description, and amount, we tweak the *filter_legitimate_txns* function to remove any element with a length of less than 4.

{% highlight ruby %}
def filter_legitimate_txns(txns):
    txns_split = txns.split("\n")
    txns_double_split = [txn.split() for txn in txns_split]
    
    return [txn for txn in txns_double_split if len(txn) >= 4]

pprint.pprint(filter_legitimate_txns(first_page_txns_raw))
{% endhighlight %}

![_config.yml]({{ site.baseurl }}/images/DBS_clean_fp_txns.png)

The process is almost the same for UOB's sample statement as it's fairly similar to DBS's, apart from three major differences. Firstly, there's an extra footnote that lies at the end of every page, so transactions are wrapped between the fixed header "PREVIOUS BALANCE" and the footnote (that starts with "Pleasenote"). Therefore, we can extract the transactions by taking whatever that comes between the header and footnote, with the following code:

{% highlight ruby %}
with pdfplumber.open(uob_source_dir / uob_pdf_file) as pdf:
    first_page = pdf.pages[0]
    first_page_text = first_page.extract_text()
    first_page_txns_raw = first_page_text.partition("PREVIOUS BALANCE")[2].partition("Pleasenote")[0]
    print(first_page_txns_raw)
{% endhighlight %}

![_config.yml]({{ site.baseurl }}/images/parsed_UOB_fp.png)

Secondly, each transaction is tagged to a unique reference number and that number appears directly below the associated transaction. We are not interested in capturing these reference numbers as they serve no purpose, so we will further edit the *filter_legitimate_txns* function to accommodate for this.

{% highlight ruby %}
def filter_legitimate_txns(txns):
    txns_split = txns.split("\n")
    txns_split_no_ref = [txn for txn in txns_split if "Ref No." not in txn] 
    txns_double_split = [txn.split() for txn in txns_split_no_ref]
    
    return [txn for txn in txns_double_split if len(txn) >= 4]

pprint.pprint(filter_legitimate_txns(first_page_txns_raw))
{% endhighlight %}

![_config.yml]({{ site.baseurl }}/images/UOB_clean_fp_txns.png)

Last but not least, UOB posts each transaction in two dates; one being the post date and the other being the transaction date. The post dates will be removed as we're only concerned with the latter.

{% highlight ruby %}
first_page_txns_processed = filter_legitimate_txns(first_page_txns_raw)
for txn in first_page_txns_processed:
    del txn[0:2]

pprint.pprint(first_page_txns_processed)
{% endhighlight %}

![_config.yml]({{ site.baseurl }}/images/UOB_processed_fp_txns.png)

So far we've managed to successfully extract all clean transactions from each bank statement's first page. However, these transactions can extend to the next if one decides to be more generous and splurge more on a given month! Therefore, we need to expand on our code base to ensure complete transaction extraction from all statements. As mentioned earlier, the end of a transactional listing can be identified by either "SUB-TOTAL" (in DBS) or "SUB TOTAL" (in UOB), so we first define a function that returns True if a page contains these and False otherwise. We shall also store the returned matched words so we can partition the transactions in *txn_trimming* function (so far we've already partitioned the transactions in the first page of each statement, and we're generalizing this operation in this function where it aims to partition transactions in all pages).

{% highlight ruby %}
sub_total_regex = re.compile("SUB.TOTAL")

def contains_sub_total(page):
    if sub_total_regex.search(page):
        return True, sub_total_regex.search(page).group()
        
    else:
        return False, None
{% endhighlight %}

{% highlight ruby %}
def txn_trimming(page_text, s):
    txns_raw = page_text.partition(s)[2]
    
    sub_total_bool, sub_total_content = contains_sub_total(txns_raw)
    
    if sub_total_bool:
        return txns_raw.partition(sub_total_content)[0]
        
    else:
        return txns_raw.partition("Pleasenote")[0]
{% endhighlight %}

With these functions we are now able to easily extract and process every transaction in all statements. However, there's a final step that we can implement to make our lives easier when we analyze the amounts later. If you look closely at the extracted transactions you would notice that some amounts are suffixed with the letters "CR". These relate to cash rebates given out by the banks because of several reasons like store refunds or monthly credit card rebates. We want the transactional amounts to only store floating-point numbers, so we need to remove the CR while adding a negative sign in front to signify a decrease in expense (instead of an increase). The following function can do this easily:

{% highlight ruby %}
def process_txn_amt(txns):
    for txn in txns:
        while not txn[-1].replace(".","",1).replace(",","",1).isdigit() and not "CR" in txn[-1]:  
            txn.pop(-1)
    
        if "CR" in txn[-1]:
            txn[-1] = txn[-1].replace("CR","",1)
            txn[-1] = "-" + txn[-1]
            
    return txns
{% endhighlight %}

When a list of transactions is passed as an argument, the function will check if each transaction ends with either a floating-point number or the letters "CR". If it doesn't, it removes the last characters until either condition is satisfied. Furthermore, for those transactions that end with the letters "CR", the letters are removed and a negative sign is added in front.

My source directory contains the entire 2020 statements from both DBS and UOB. Putting all the pieces together, running the above codes on all the files will give all 2020 transactions:

{% highlight ruby %}
dbs_all_txns = []
uob_all_txns = []

for folder, subfolder, pdf_files in os.walk(dbs_source_dir):
    for pdf_file in pdf_files:

        with pdfplumber.open(dbs_source_dir / pdf_file) as pdf:
            for i in range(2):  # txns only extend up to 2nd page
                    page_text = pdf.pages[i].extract_text()
                    sub_total_bool, sub_total_content = contains_sub_total(pdf.pages[0].extract_text())

                    if i == 0:
                        txns_raw = txn_trimming(page_text, "NEW TRANSACTIONS JEROME KO JIA JIN")
                        dbs_all_txns.append(process_txn_amt(filter_legitimate_txns(txns_raw)))

                    elif i == 1 and not sub_total_bool:  # if txns extend to 2nd page
                        txns_raw = txn_trimming(page_text, "2 of 3")
                        dbs_all_txns.append(process_txn_amt(filter_legitimate_txns(txns_raw)))

for folder, subfolder, pdf_files in os.walk(uob_source_dir):
    for pdf_file in pdf_files:

        with pdfplumber.open(uob_source_dir / pdf_file) as pdf:
            for i in range(2):  # txns only extend up to 2nd page
                    page_text = pdf.pages[i].extract_text()
                    sub_total_bool, sub_total_content = contains_sub_total(pdf.pages[0].extract_text())

                    if i == 0:
                        txns_raw = txn_trimming(page_text, "PREVIOUS BALANCE")
                        uob_all_txns.append(process_txn_amt(filter_legitimate_txns(txns_raw)))

                    elif i == 1 and not sub_total_bool:  # if txns extend to 2nd page
                        txns_raw = txn_trimming(page_text, "Date Date SGD")
                        uob_all_txns.append(process_txn_amt(filter_legitimate_txns(txns_raw)))

for monthly_txns in uob_all_txns:
    for txn in monthly_txns:
        del txn[0:2]  # remove post dates

all_txns = dbs_all_txns.copy()
all_txns.extend(uob_all_txns)

categorized_txns = [{"Date": " ".join(txn[0:2]), "Txn Desc": " ".join(txn[2:len(txn)-1]), "Amt": txn[-1]}
                        for monthly_txns in all_txns 
                        for txn in monthly_txns]

{% endhighlight %}

![_config.yml]({{ site.baseurl }}/images/categorized_txns.png)

## Write Into .csv
The above transactions are first converted into a pandas dataframe to allow for easier further manipulation to the data and subsequent writing into a csv file.

Adding cateogories to each transaction then requires the following code:

{% highlight ruby %}
# Load into dataframe for further manipulation
df_categorized_txns = pd.DataFrame(categorized_txns)

# Format date column
df_categorized_txns["Date"] = df_categorized_txns["Date"] + " 2020"

# Categorizing txns
shopping = re.compile(r'''.*Shopee(pay)?.*
                    |.*Lazada.*
                    |.*Zalora.*
                    |.*Uniqlo.*
                    |.*Asos.*
                    |.*Ghbass.*
                    |.*Decathlon.*
                    |.*Amazon.*
                    |.*Watson.*
                    |.*Guardian.*''', re.I | re.VERBOSE)
utilities = re.compile(r".*Liberty Wireless.*|.*Gomo.*", re.I)
holiday = re.compile(r".*Agoda*.|.*Scoot.*|.*Hotel.*", re.I)
grooming = re.compile(".*Sultans of shave.*", re.I)
entertainment = re.compile(r".*GV.*|.*Shaw.*", re.I)
others = re.compile(r".* Fee.*|.*Charge.*|.*Interest.*|.*Bank.*|.*Rebate.*", re.I)

def categorize_txns(s):
    if shopping.search(s["Txn Desc"]):  # None if there's no match
        return "Shopping"
    
    elif utilities.search(s["Txn Desc"]):
        return "Utilities"
    
    elif holiday.search(s["Txn Desc"]):
        return "Holiday"
    
    elif grooming.search(s["Txn Desc"]):
        return "Grooming"
    
    elif entertainment.search(s["Txn Desc"]):
        return "Entertainment"
    
    elif others.search(s["Txn Desc"]):
        return "Others"
    
    elif s["Date"] == "27 JUN 2020":
        return "Birthday"
    
    elif s["Date"] == "19 MAR 2020":
        return "Anniversary"
    
    else:
        return "Food"

df_categorized_txns["Category"] = df_categorized_txns.apply(categorize_txns, axis=1)
{% endhighlight %}

This part is rather tedious; it requires one to study the underlying data and understand which merchants one tends to gravitate to fulfill most of the purchases in each category. For example, I know that I mainly shop from ecommerce channels such as Lazada and Shopee so those are used in the regex functions. In addition, I've allocated budget for my partner's birthday so the above attempts to classify all 27 Jun transactions as expenses incurred for her birthday.

Lastly, the dataframe is written into a csv file to prepare for analysis:
{% highlight ruby %}
df_categorized_txns.to_csv(dest_csv / "2020 transactions.csv")
{% endhighlight %}
## Analyze Expenses
We finally have a clean dataset to analyze the expenses. The resulting csv file is uploaded in Tableau Public to generate the dashboard [here](https://public.tableau.com/profile/jerome.ko#!/vizhome/2020ExpenseAnalysis/Dashboard1?publish=yes).

![_config.yml]({{ site.baseurl }}/images/expense_dashboard.png)

<!-- 
Talk about budget first
Check month on month expenditure and count of transactions (on those periods of high txns, do we see high amt too?)
Check counts of each category per month (interesting to see which months are the most fun by looking at count of entertainment)
Check highest occurrence for each string
Cumulative line graph for each category vs benchmark/budget
Monthly growth rate

Interesting findings:
1. Seemingly healthy, positive balance: Spent $891 less than total budget; this is not great news esp when one should be saving more during COVID!
2. Above is bad too because we're not considering non-card transactions - Expense is prob more
3. Txn freq doesn't correlate to total txn amt; holiday season saw the highest transactional count but amt is fairly stable around monthly average
4. Breaking budget and expenses down into categorical level proves to be very helpful; even though overall balance is negative we can clearly see that we're spending too much on food; 2x more than allocated budget!
5. May is the season of spending; inflated because bought a personal Macbook for myself. Perhaps still the best investment made as I've been using it to hone my programming skills

Takeaway:
1. Try to contain all transactions in cards, so tracking is more accurate
 -->

<!-- Code can be found here.
Tableau dashboard can be found here. -->