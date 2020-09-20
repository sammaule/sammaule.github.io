---
layout: post
title: "I am the walrus"
date: 2020-09-20
---

In Python 3.8 an assignment expression (:=) known as the walrus operator was introduced. This is a neat operator that can help make your Python code more succint, reducing unecessary duplication, making it easier to read, a tip that I picked up from the excellent (so far) [Effective Python](https://smile.amazon.co.uk/Effective-Python-Specific-Software-Development/dp/0134853989/ref=sr_1_1?crid=TNU2EBTVHGHT&dchild=1&keywords=effective+python&qid=1600607285&sprefix=effective+py%2Caps%2C149&sr=8-1) book.

The walrus operator allows you to assign variables in places where you're not normally allowed - like in an if statement. For example, a recent example that I had implemented in a python front end, was concerned with whether a risk score was in range(6). I had written something like:

```python
data = {"risk": 2}

risk = data.get("risk")
if risk in range(6):
    show_risk_score(risk)
else:
    raise ValueError("risk score should be in range(6)")
```

A simpler implementation using the walrus operator would be:

```python
if (risk := data.get("risk")) in range(6):
    show_risk_score(risk)
else:
    raise ValueError("risk score should be in range(6)")
```

Now the focus is taken away from the assignment of the risk variable. Note the need for the parenthesese around the walrus operator here. These are not required for a simple zero check, for example:

```python
if risk := data.get("risk"):
    show_risk_score(risk)
else:
    raise ValueError("risk score shouldn't be zero")
```

Walrus operators can also make switch case statements (statements that help to control the flow of the programme depending on the state of some variables).

For example, say you are building a simulation of a hospital and the patients are given drugs for a condition in some order of efficacy, where the most effective drugs are always administered if they are in stock, and if they are not the next best are given. In this case, we may have written the implementation like this:

```python
drug = drugs.get("drug_a", 0)
if drug:
    administer_drug(drug)
else:
    drug = drugs.get("drug_b", 0)
    if drug:
        administer_drug(drug)
    else:
        drug = drugs.get("drug_c", 0)
        if drug:
            administer_drug(drug)
        else:
            discharge_patient()
```

This is code is quite hard to follow, but common in Python as there wasn't a particularly clean implementation, until the walrus operator arrived. Now this logic can be written in a much more readable way:

```python
if drug := drugs.get("drug_a", 0):
    administer_drug(drug)
elif drug := drugs.get("drug_b", 0):
    administer_drug(drug)
elif drug := drugs.get("drug_c", 0):
    administer_drug(drug)
else:
    discharge_patient()
```

There are many other use cases for the walrus operator in Python, but these are the two that I can see myself using most often when using the language. Given the clear improvements to readability I'll be making the switch to the walrus operator when using Python 3.8 and above.
