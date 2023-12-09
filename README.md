# AOAI-Monitoring-Load-balancing-with-APIM

The advent of Generative AI has created many use cases and scenarios for organizations to bring in additional efficiency in their businesses and enhance the user experience provided by their applications. Azure’s OpenAI service has generated considerable interest from enterprises in their quest to realize this potential offered by Generative AI technology. 
Introducing any new technology into an enterprise setup, however, calls for an increased focus on areas around security and governance. Azure OpenAI (AOAI) Service provides REST API access to OpenAI's language models including the GPT-4, GPT-35-Turbo, and Embeddings models and these APIs can be consumed from Enterprise applications. Here, Azure API Management (APIM) can be brought in to provide enhanced capabilities around Security, Monitoring, Load Balancing, Rate limiting & throttling and overall governance when using AOAI APIs, these points have been explained in detail through multiple blogs and Azure documentation like the ones here and here. 
AOAI processes text by breaking it down into tokens. Tokens can be words or just chunks of characters. For example, the word “Welcome” gets broken up into the tokens “wel”, and “come”, while a short and common word like “door” is a single token. The total number of tokens processed in each request depends on the length of your request prompt, response from AOAI and request parameters. The pricing of AOAI APIs is dependent on the total count of tokens that are consumed by applications, hence organizations have shown interest in being able to track the consumption of tokens on a per application basis, this will enable organizations to understand the usage of AOAI APIs across the consuming applications and potentially create a charge back model for such usage. APIM when used as a gateway to access AOAI can enable such tracking of token consumption across consuming applications, let’s look at how this can be achieved.
We can consider two approaches for measuring the token consumption at APIM. The first option would be to use the unique subscription key per consuming application. Developers who need to consume published APIs must include a valid subscription key in HTTP requests when calling those APIs, we can then log the subscription keys and the total tokens consumed when calling AOAI APIS to track the usage. But many organizations consider securing APIs using OAuth2 token as a more standard approach which we can consider as the second option. The OAuth2 token generated from Azure AD takes the form a JWT (JSON Web Token) which is a JSON payload that is signed with the private key of the issuer, and can be parsed and verified by APIM. This token comprises of many fields, among them, “sub” represents the principal about which the token asserts information such as the user of the app and this can be used by us to measure token consumption. In the next section we will look at how to measure token consumption based on the “sub” field in the JWT token. 
The focus of this repo is on the APIM policies that can help with tracking AOAI token usage based on the sub field in the JWT token of consuming application, this scenario has been depicted below:
![OpenAI Token Count with APIM](https://github.com/sdadha/AOAI-Monitoring-Load-balancing-with-APIM/assets/92512653/7767f492-3022-468b-8724-1181d400f196)

 
 
•	Secure access to OpenAI API in APIM by adding “validate-jwt” policy in the inbound processing section.
![image](https://github.com/sdadha/AOAI-Monitoring-Load-balancing-with-APIM/assets/92512653/8087fba2-b0e6-4f0a-a713-99e94346b7d2)

 

•	The “validate-jwt” token policy can be configured to capture the token value in a variable mentioned as part of the “output-token-variable-name” configuration:
![image](https://github.com/sdadha/AOAI-Monitoring-Load-balancing-with-APIM/assets/92512653/15e77741-20d1-4731-bec6-3b8670e7a6aa)

 
•	APIM supports fetching of values from within the JWT token using policies, the policy snippet below fetches and sets the “subject” from JWT Token as a header named “caller-appid”, this policy snippet can be placed in the <inbound> section of APIM polcies
![image](https://github.com/sdadha/AOAI-Monitoring-Load-balancing-with-APIM/assets/92512653/edf6db02-384a-46ec-99b9-a256d18736ad)

 
•	We can use this field to track OpenAI API invocations from specific principal/applications.
•	The response from AOAI contains information related to token usage including prompt tokens, response tokens and total tokens.
•	Using the code snippet below, it is possible to fetch the total tokens and set the same as header:
![image](https://github.com/sdadha/AOAI-Monitoring-Load-balancing-with-APIM/assets/92512653/b66dd55d-3ce4-4199-859b-8254484b2084)

 
•	The above code snippet needs to be placed in the outbound section of APIM policies, as the total tokens field will be in response from AOAI.
•	To be able to log these headers in Azure Monitor we need to configure the diagnostic settings in the API Management instance and create a setting that points to a log analytics workspace:
![image](https://github.com/sdadha/AOAI-Monitoring-Load-balancing-with-APIM/assets/92512653/c9ed6ab4-5cb9-48b1-9b41-327cb7a0cff1)
 


•	We can now configure the AOAI API in APIM to log these headers in Azure Monitor, click on the settings tab of the API:
![image](https://github.com/sdadha/AOAI-Monitoring-Load-balancing-with-APIM/assets/92512653/79293229-c4dd-4272-ab4d-32c92ca8dd79)
 

•	Under the settings tab, click on Azure Monitor and set sampling to 100%
![image](https://github.com/sdadha/AOAI-Monitoring-Load-balancing-with-APIM/assets/92512653/281cad1d-cbe8-4b3f-aa38-04b7cb87dda2)

 
•	Under additional settings mention the headers to log: “total-tokens” from the front-end response and “caller-subject” from the backend request.
![image](https://github.com/sdadha/AOAI-Monitoring-Load-balancing-with-APIM/assets/92512653/3c7a2e45-fc3e-403f-92d3-581b7a47cd9f)

 
•	With the above setting every request for this API will lead to logging of the subject from JWT token and the corresponding AOAI token consumption for that request in Azure Monitor – Log Analytics Workspace. This data will be available in the APIManagementGatewayLogs table.
•	To fetch these details, we can run KQL queries like the one below:
![image](https://github.com/sdadha/AOAI-Monitoring-Load-balancing-with-APIM/assets/92512653/0a026aed-4016-44e3-bddc-0ade5955d9b3)
 


We can use this data to generate dashboards or create a charge back/billing mechanism for consuming applications. Apart from Azure Monitor, we also have options like using Event Hub or connecting to partner logging solutions to send this data. This blog is a small example of how placing APIM as a gateway in front of AOAI can helps us secure,  monitor, manage and govern generative AI projects.


