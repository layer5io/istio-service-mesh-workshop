# Optional Lab 2 - Setting up API Tokens (optional)

As part of this workshop we will be sending access logs from Istio to [Loggly](https://www.loggly.com/) and telemetry data to [Appoptics](https://www.appoptics.com/). In this lab we will take you through the steps needed to setup Loggly & Appoptics API tokens and configure Appoptics dashboard.

## Steps

* [1. Set up loggly API Token](#1)
* [2. Setup Appoptics API Token](#2)

## <a name="1"></a> 1 - Set up Loggly API Token


Getting a Loggly API token for use with istio:

[Loggly](https://www.loggly.com/)
![](img/loggly.png)

To signup for [Loggly signup](https://www.loggly.com/signup/)
![](img/loggly_signup.png)

[Loggin sign in](https://www.loggly.com/login/)
![](img/loggly_signin.png)

After login you will be taken to the Loggly landing page. From here, select `Source Setup` from the top menu: ![](img/loggly_landing_page.png)

On the `Source Setup` page select the `Customer Tokens` item from the sub-menu
![](img/loggly_source_setup.png)


On the `Customer Tokens` page let us create a new Token by using the `Add New` button.
![](img/loggly_customer_token.png)

This open a popup confirming the creation of a new token.
![](img/loggly_new_customer_token.png)

Please enter a good description for you to identify the token.

Once the token is created you will be able to see it in the Tokens table. On the left there is a `copy to clipboard` button which can help with copying the new token.
![](img/loggly_new_token.png)

We can now store this token in an environment variable for later use:
```sh
LOGGLY_TOKEN="PASTE YOUR TOKEN HERE"
```


## <a name="2"></a> 2 - Setup Appoptics API Token
Reserve your lab account [here](https://docs.google.com/spreadsheets/d/174haSpPTlDZeZLJTJRUeHZydgxpnTQScthtLLWMj3mc/edit?usp=sharing).

Let us store reserved token in an environment variable for later use:
```sh
AOTOKEN="PASTE YOUR TOKEN HERE"
```