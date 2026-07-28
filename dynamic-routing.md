
It is important to understand JusPay's implementation of dynamic routing to be able to configure gateway priorities correctly for your Merchant Account. Our approach has been to ensure maximum conversion rate and the implementation reflects this. The dynamic routing algorithm uses a **non-deterministic mathematical model**  to rank all applicable gateways for the given transaction, using a lot of information, such as gateway health, gateway performance for a similar transaction recently, historical performance, etc.. Along with this, we take into account the priorities configured by you and arrive at the final gateway, such that the success probability is the highest.

So, please do not be surprised if a different gateway is chosen for a particular transaction which doesn't really align with your configuration. Getting you the maximum success rate is more important to us.

If a particular gateway is not applicable for a transaction, say payment mode is SBI NetBanking and gateway is HDFC PG, then that gateway will be automatically dropped by us. So, you don't have to take care of that in the priority logic.


## Simple routing



Most typical routing logic would be to set up one gateway as primary and another gateway as secondary. Secondary gateway is usually used as a backup in case of primary gateway going down. However, there could be cases where a specific card BIN performs really well with the given PG and we might end up choosing the secondary gateway in such a case.

You can configure the priorities of gateways using a comma-separated string representation. For instance, say you have two gateways namely HDFC and ICICI. Now, if you need to setup HDFC as primary and ICICI as secondary, then you should configure the priorities as given below:

`HDFC,ICICI`


## DSL



We understand that one size doesn't fit all. So, we have come up with another flexible option where you have the complete capability to dynamically set the priority.

You can describe the priority logic in Groovy code and set the priority dynamically for any given transaction. The gateways should be identified using the unique names (enumerations given below) and you are expected to setup the priority in the form of a string List. First gateway in the list has the highest priority and so will be preferred over others.

**Note: Good code writing skills are necessary here. If you dont know what this is or is doubtful, please get in touch with us along with your requirement and we will help set this up for you.** 

**Context** 

Transaction details are given as input to the DSL. Please see below for all the input variables that will be available to you in the DSL.


#### Java Code Snippet:

```java
def order = [orderId: "orderId", amount: 5000.00, currency:"INR", preferredGateway: "AXIS", 
              udf1:"web", udf2: "desktop", udf3: "flight_booking", udf4: "", udf5: "", 
              udf6: "", udf7: "", udf8: "", udf9: "", udf10: ""]

def txn = [ txnId: "txnId", isEmi: true, emiBank: "HDFC", emiTenue: 6 ]

def payment = [cardBin: "524368", cardIssuer: "HDFC Bank", cardType: "CREDIT", cardBrand: "MASTERCARD", 
                paymentMethod: "MASTERCARD", paymentMethodType: "CARD", cardIssuerCountry: "INDIA", authType:"OTP"]

// use the below for randomization
long currentTimeMillis = _milliseconds_since_epoch_in_UTC;

// all the above variables are available to the DSL
```


**Example 1. Routing based on Channel** 


#### Java Code Snippet:

```java
 def priorities = ['HDFC', 'ICICI', 'PAYU'] 
  // above is the default priority

  if (order.udf1 == 'web') {
    priorities = ['HDFC','PAYU','ICICI']
  }
  else if (order.udf1 == 'mobile' && order.udf2 == 'android')
    priorities = ['ICICI','HDFC','PAYU']
  }
  setGatewayPriority(priorities)
```


**Example 2. Volume based routing** 


#### Java Code Snippet:

```java
  // def myGateways = ['HDFC','ICICI']
  // Goal: 50% approx split between two gateways

  if(currentTimeMillis % 10 < 5) {
    setGatewayPriority(['HDFC', 'ICICI'])
  }
  else {
    setGatewayPriority(['ICICI', 'HDFC'])  
  }

  // def myGateways = ["HDFC","ICICI", "AXIS"]
  // Goal: 1/2 split between two gateways and use third as backup
  if(currentTimeMillis % 10 < 5) {
    setGatewayPriority(['HDFC', 'ICICI', 'AXIS'])
  }
  else {
    setGatewayPriority(['ICICI', 'HDFC', 'AXIS'])
  } 
```


**Example 3. Issuer based routing** 


#### Java Code Snippet:

```java
  def priorities = [ 'HDFC', 'ICICI' ] 
  // default priorities
  if (payment.cardIssuer == 'ICICI Bank') { 
    // if ICICI Bank card, use ICICI
    priorities = [ 'ICICI', 'HDFC' ]
  }
  else if (order.udf1 == 'mobile' && order.udf2 == 'android')
    // for android transactions, use ICICI
    priorities = [ 'ICICI', 'HDFC' ]
  }
  else { // for everything else use HDFC as primary
    priorities = [ 'HDFC', 'ICICI' ]
  }
  setGatewayPriority(priorities) 
```


**Example 4. Card Brand based routing** 


#### Java Code Snippet:

```java
  def priorities = ['HDFC', 'ICICI', 'PAYU'] 
  // default priorities
  if (payment.cardBrand == 'MAESTRO') { 
    // if Maestro card, use ICICI as primary
    priorities = ['ICICI', 'PAYU', 'HDFC']
  }
  else if (payment.cardBrand == 'AMEX')
    priorities = ['PAYU', 'ICICI', 'HDFC']
  }
  setGatewayPriority(priorities)
```



## **Enforce Gateway Routing** 



The Enforce gateway routing can be utilised if you want to route certain set of orders/transactions to a particular payment gateway irrespective of gateway health, performance etc. For example when an offer is being provided by a certain aggregators then you would want to route transactions to that particular aggregator.

Function to be used in PL for enforcement is:`enforceGatewayPriority(["gatewayName"]);`


#### Java Code Snippet:

```java
  def priorities = ["AXIS", "PAYU"] 
if (order.udf1 == "payu_offer") {
    priorities = ["PAYU"]
    enforceGatewayPriority(priorities)
}
else {
    setGatewayPriority(priorities)
}
```


In the above case if udf1 is passed as payu_offer then the transaction is enforced to Payu irrespective of health, score etc. Else the transaction can get routed to either Axis or Payu.


| Gateway | Gateway ID |
|---|---|
| AXIS | 1 |
| HDFC | 2 |
| ICICI | 3 |
| CITI | 4 |
| AMEX | 5 |
| CYBERSOURCE | 6 |
| IPG | 7 |
| MIGS | 8 |
| KOTAK | 9 |
| EBS | 11 |
| PAYU | 12 |
| ATOM | 15 |
| CCAVENUE_V2 | 16 |
| TPSL | 17 |
| PAYTM | 18 |
| PAYTM_V2 | 19 |
| PAYPAL | 20 |
| HDFC_EBS_VAS | 21 |
| PAYLATER | 22 |
| RAZORPAY	 | 23 |
| FSS_ATM_PIN | 24 |
| EBS_V3	 | 25 |
| ZAAKPAY | 26 |
| BILLDESK | 27 |
| SODEXO | 28 |
| BLAZEPAY | 29 |
| FSS_ATM_PIN_V2 | 30 |
| MOBIKWIK | 31 |
| OLAMONEY | 32 |
| FREECHARGE	 | 33 |
| MPESA | 34 |
| SBIBUDDY | 35 |
| JIOMONEY | 36 |
| AIRTELMONEY | 37 |
| AMAZONPAY | 38 |
| PHONEPE | 39 |
| OLAPOSTPAID | 40 |
| SIMPL | 41 |
| GOOGLEPAY	 | 42 |
| FREECHARGE_V2 | 43 |
| ITZCASH | 44 |
| TATAPAY | 46 |
| CRED | 49 |
| STRIPE | 50 |
| GOCASHFREE | 70 |
| FSSPAY | 73 |
| PINELABS | 74 |
| PAYFORT | 75 |
| MPGS | 77 |
| ONEPAY | 79 |
| EASEBUZZ | 80 |
| NOON | 81 |
| TWID | 82 |
| PAYGLOCAL | 83 |
| WORLDPAY | 87 |
| DUMMY	 | 100 |
| HDFC_IVR | 201 |
| CAPITALFLOAT	 | 260 |
| EPAYLATER | 251 |
| LAZYPAY | 252 |
| BAJAJFINSERV | 254 |
| LOANTAP | 255 |
| SHOPSE | 256 |
| SEZZLE | 258 |
| ICICIPAYLATER	 | 259 |
| HDFC_FLEXIPAY | 261 |
| MOBIKWIKZIP | 262 |
| SNAPMINT | 263 |
| AXISNB | 300 |
| HDFCNB | 301 |
| ICICINB | 302 |
| TPSL_SI | 400 |
| AXIS_UPI | 500 |
| HDFC_UPI	 | 501 |
| INDUS_UPI | 502 |
| KOTAK_UPI	 | 503 |
| SBI_UPI | 504 |
| ICICI_UPI | 505 |
| VIJAYA_UPI	 | 506 |
| HSBC_UPI | 507 |
| YESBANK_UPI	 | 508 |
| PAYTM_UPI | 509 |
| AXIS_BIZ | 511 |
| CAMSPAY | 513 |
| YES_BIZ | 514 |
| LINEPAY | 600 |
| CHECKOUT | 601 |
| CASH | 700 |
| MORPHEUS | 800 |
| ADYEN | 1001 |
| BRAINTREE | 1002 |
| TRUSTPAY | 1004 |
| POSTPAY | 1006 |


List of card brands:

* VISA
* MASTERCARD
* MAESTRO
* RUPAY
* AMEX
* DINERS
* DISCOVER

List of card types:

* CREDIT
* DEBIT


## Configure priority



To set this up, please proceed to [Gateways](https://dashboard.expresscheckout.juspay.in/) and enable the checkbox that says: I'd rather write code to define the priority. In the given text area, enter your DSL logic and then click on **Update Gateway details** .


## Special rules



In addition to the routing logic explained above, we have also provided you with the ability to define specific rules to assign different gateway priorities for a given card BIN or card issuer bank. For instance, if you have negotiated special rates for HDFC cards on HDFC PG, then you can configure this on the dashboard.

**ISIN & Issuer rules** 

ISIN stands for **Issuer Identification Number**  which is card BIN (Bank Identification Number). For, a given ISIN and given Gateway, you can setup a priority score. If you wish to setup a rule for a particular card issuer bank, that can be configured too.


| ISIN	 | Gateway | Score |
|---|---|---|
| 524368 | HDFC | 1.1 |
| 447746 | ICICI | 1.1 |


**Score** 

The score is expected to be in the range of **0.0 - 2.0** (decimals are allowed). Giving a value of 2.0 will increase the priority of the gateway by 2x. A value of 0.5 will reduce the priority score by 50%. A score of 1.0 will make the rule neutral.

**Issuer Routing** The issuer typically refers to the bank which has issued the credit or debit card. In some countries, even credit unions or prepaid card providers can also issue cards. You can set up rules to specify dynamic scoring for a particular issuer and a gateway

**Card Brand Routing** Card Brands include VISA, MASTERCARD, MAESTRO, AMEX, RUPAY, DISCOVER, DINERS, JCB. Setup your scoring to route based on a card brand and a gateway


## Dynamic Routing based on High Failures



AI-ML algorithm constantly monitor and learn high payment failures across payment ecosystem along with merchant-context and make fully automatic routing decisions

* **Granular** : Router takes into account high payment failures of all possible combinations of payment gateways and payment methods
* **Platform level** : Looks at global information on the Juspay platform across merchants
* **Real-time** : Identifies potential downtimes in near real-time, many times within a minute
* **Accurate** : Looks at a series of payment failures consecutively across board yield very high accuracy
* **Contextual** : When downtimes are not global and very much specific to the merchant level, Router uses merchant-level patterns and makes routing decisions
* **Configurable** : You configure thresholds of failures at your merchant account level, payment method level, and enable/disable payment methods to be a part of the routing algorithm. Works in conjunction with, health-based routing and static routing rules

**Score** The score is expected to be in the range of **0 to 1** . Algorithm maintains the score **per gateway and payment method**  combination per 30 minute time-window. It reduces score when transaction starts and increase scores when transaction succeeds.

Whenever score falls below configured threshold, gateway decider service would look for another gateways supporting that payment method with better score.



---

## See Also

- [Payment Form](https://juspay.io/in/docs/add-ons/docs/payment-forms/payment-form)
- [Overview](https://juspay.io/in/docs/add-ons/docs/payment-links/overview)
