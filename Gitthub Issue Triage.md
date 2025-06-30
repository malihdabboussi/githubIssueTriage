
# **N8N Github Issue Triage Challenge**

This tool to build AI agent for Github issue triage, the goal of this tool is to 


AI-Powered GitHub Issue Triage System. The goal is to create n8n workflow that fetches new GitHub issues and enriches them with an AI-generated summary and assigns them to the correct internal team (e.g., Engineering, Sales, Support). The final, enriched data will be stored in Supabase.


# **AI Models**

In this Challenge, I used two AI models.
### **Mistral-Medium-latest Model**
**Used for:**
* **categorization**
* **Assigned Role**
* **Sentiment**

#### **Prompt**

The prompt consists of the output format and rules, and supported with example.


```
Analyze the GitHub issue object provided, summarize clearly in one sentence, and generate relevant tags based on context.

Do NOT include markdown, extra characters, or explanations.
Provide your response strictly in this JSON format, without any markdown or additional characters:

{
  "url": "{{$json.url}}",
  "summary": "One-sentence summary",
  "tags": ["tag1", "tag2", "tag3"],
  "user_login": "{{$json.user_login}}",
  "body": "{{$json.body.replaceSpecialChars()}}",
  "category": "Bug" | "Feature Request" | "Question" | "Undetermined",
  "assigned_role": "BUG or Feature Request  ='Engineering'",  " For inqueries about price demos or business = 'Sales' " , "for how to questions or user guidance = 'Support' ",
  "title": "{{$json.titel}}",
  "sentiment": "Positive" | "Negative" | "Neutral" //Analyze "sentiment" based on the {{$json.body}} text.
}

Example Input:
{
  "body": "breaks hmr and causes some duplicate react instances (it was also supposed to fix some of those, seems neither version is correct) Closes WEB-1882"
}

Example Output:

{
  "summary": "Fixes Hot Module Replacement (HMR) issue causing duplicate React instances.",
  "tags": ["hmr", "react", "bug", "fix"],
  "sentiment": "Negative"
}
```



### **Mistral-Small Model**
**Used For:**

* output parser, connected to output parser node, to reformat the response/result comming from the main model.

## **Helpers**
### **Response Handler Code**
Cleaning **HTTP response**, the first code cleaned the response comming from HTTP request and removed unwanted characters by using main regex setting cleaner.

```
const cleanString = (input) => {
  return input
    .replace(/<details>[\s\S]*?<\/details>/gi, '') // Remove <details> sections
    .replace(/<[^>]*>/g, '')  // Remove HTML tags
    .replace(/\\[nrt"\\]/g, ' ')  // Replace escaped \n, \r, \t, \", \\ with space
    .replace(/[\n\r\t]+/g, ' ') // Replace real newlines, tabs, returns with space
    .replace(/\s{2,}/g, ' ')  // Condense multiple spaces into one
    .trim();   // Remove leading/trailing whitespace
};
```

then extract the needed format:

```
const escapeJson = (str) => {
  return JSON.stringify(str).slice(1, -1);
};

for (const item of $input.all()) {
  new_outputs.push({
    url: escapeJson(item.json.url),
    user_login: escapeJson(item.json.user.login),
    body: escapeJson(cleanString(item.json.body)),
    titel: escapeJson(item.json.title),
  });
}
```

### **Output Mapper**
This small code used to map out the output values coming from the AI results. the output parser put the results in output key attribute, and needed this code to extract the output value final format to be added to Supabase database.

# **Bonus Implementation**

### **Tackled bonuses**
* Sentiment Analysis.
* Dynamic Tagging.
* Smart Notifications.


### **Table Altering**
the **enriched_issues** was altered and add two extra columns:
```
alter table enriched_issues
add column sentiment varchar(50);

alter table enriched_issues
add column tags varchar[] default '{}';
```

### **Adding Switch and Discord webhook Nodes**

Finally, **Switch Node** used be do the proper routing based on assigned_role, the node added after the suppabase node to make sure the flow was executed correctly before routing and sending webhooks.

I am more familiar with discord, so i used discord nodes, created [**wzTechnology server**](https://discord.gg/E9EFpN6h) with channels:
* [Sales](https://discord.com/channels/1388885264637759660/1388885292454514738)
* [Engineering](https://discord.com/channels/1388885264637759660/1388885317276270644)
* [Support](https://discord.com/channels/1388885264637759660/1388885354102128733)


# **Note**
The flow tested with issue that has more than one issue assigned in a single request.

Tested Issues: 
* https://api.github.com/repos/invertase/react-native-notifee/issues/15 
* https://api.github.com/repos/Azure/Azure-Functions/issues/1457 
* https://api.github.com/repos/danny-avila/LibreChat/issues/1267 
* https://api.github.com/repos/Expensify/App/issues/61747 
















 
