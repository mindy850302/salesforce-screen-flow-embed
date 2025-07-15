# Embedding Salesforce Screen Flows in External Websites

This guide explains how to embed Salesforce Screen Flows into external websites, allowing you to leverage the power of Salesforce flows outside of the Salesforce platform.

## Table of Contents

- [What are Salesforce Screen Flows?](#what-are-salesforce-screen-flows)
- [Prerequisites](#prerequisites)
- [Step-by-Step Implementation Guide](#step-by-step-implementation-guide)
- [Code Example](#code-example)
- [Advanced Configuration Options](#advanced-configuration-options)
- [Troubleshooting](#troubleshooting)

## What are Salesforce Screen Flows?

Salesforce Screen Flows are a powerful point-and-click tool in Salesforce that allow you to build complex business processes with a user interface. They provide a guided experience that can collect and display information, create and update Salesforce records, and integrate with other systems.

By embedding screen flows in external websites, you can:

- Provide consistent user experiences across platforms
- Leverage Salesforce's powerful automation capabilities outside of Salesforce
- Collect data directly into Salesforce from external sites
- Maintain a single source of truth for business logic

## Prerequisites

Before you begin, ensure you have the following:

1. **Salesforce Experience Cloud Site**: You need a Salesforce Experience Cloud (formerly Community) site where your flow will be hosted
2. **Screen Flow**: A working screen flow built in Salesforce Flow Builder
3. **Lightning Component**: A Lightning component to host your flow (typically a Lightning app and component pair)
4. **Proper Permissions**: Necessary permissions set up in Salesforce to allow external access

## Step-by-Step Implementation Guide

### 1. Create Your Screen Flow in Salesforce

1. In Salesforce Setup, navigate to **Process Automation** > **Flows**
2. Create a new flow or select an existing one
3. Ensure your flow type is **Screen Flow**
4. Build your flow with the desired screens and logic
5. Activate your flow

### 2. Create a Lightning App for Embedding

1. In your Salesforce Developer Console, create a new Lightning Application:

   ```javascript
   <!-- embeddedFlowFormResponseSubmissionApp.app -->
   <aura:application access="GLOBAL" extends="ltng:outApp">
       <aura:dependency resource="c:embeddedFlowFormResponseSubmission"/>
   </aura:application>
   ```

2. Create a Lightning Component to host the flow:

   ```javascript
   <!-- embeddedFlowFormResponseSubmission.cmp -->
   <aura:component implements="flexipage:availableForAllPageTypes" access="global">
       <lightning:flow aura:id="flowData" onstatuschange="{!c.statusChange}"/>
       
       <aura:handler name="init" value="{!this}" action="{!c.init}"/>
   </aura:component>
   ```

3. Create the JavaScript controller for your component:

   ```javascript
   // embeddedFlowFormResponseSubmissionController.js
   ({
       init : function (cmp) {
           // Get the flow API name from your Salesforce Flow
           var flow = cmp.find("flowData");
           flow.startFlow("Your_Flow_API_Name");
       },
       
       statusChange : function (cmp, event) {
           if(event.getParam("status") === "FINISHED") {
               // Handle flow completion
           }
       }
   })
   ```

### 3. Configure Your Experience Cloud Site

1. Navigate to **Setup** > **Digital Experiences** > **All Sites**
2. Select your Experience Cloud site or create a new one
3. Ensure your site is active and published
4. Add your Lightning app to the site

### 4. Enable Lightning Out in Your Experience Site

1. Navigate to **Setup** > **Session Settings**
2. Ensure that **Lightning Out** is enabled
3. Add any necessary domains to the **CORS Allowed Origins List** in **Setup** > **CORS**

### 5. Embed the Flow in Your External Website

Add the following code to your HTML page:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Embedded Flow Page</title>
</head>
<body onload="init()">
    <h1>Embedded Salesforce Flow</h1>
    <div id='container'></div>
    
    <!-- Replace with your Experience Cloud site URL -->
    <script type='text/javascript' src="https://your-experience-site.force.com/EmbeddedFlow/lightning/lightning.out.js"></script>
    
    <script type='text/javascript'>
        function init() {
            $Lightning.use(
                "c:embeddedFlowFormResponseSubmissionApp",  // name of the Lightning app
                function() {  // Callback once framework and app loaded
                    $Lightning.createComponent(
                        "c:embeddedFlowFormResponseSubmission",  // top-level component of your app
                        {},  // attributes to set on the component when created
                        "container",  // the DOM location to insert the component
                        function(cmp) {  // callback when component is created and active on the page
                            console.log("Flow component created successfully");
                        }
                    );
                },
                'https://your-experience-site.force.com/EmbeddedFlow'  // Experience Cloud site endpoint
            );
        }
    </script>
</body>
</html>
```

## Advanced Configuration Options

### Passing Parameters to Flows

You can pass input variables to your flow by adding them to the attributes object in the `$Lightning.createComponent` function:

```javascript
$Lightning.createComponent(
    "c:embeddedFlowFormResponseSubmission",
    {
        inputVariables: [
            {
                name: "recordId",
                type: "String",
                value: "001XXXXXXXXXXXXXXX" // Can be dynamically set
            }
        ]
    },
    "container",
    function(cmp) {}
);
```

### Styling the Embedded Flow

You can style your embedded flow using CSS. Create a style section in your HTML or link to an external CSS file:

```html
<style>
    #container {
        max-width: 800px;
        margin: 0 auto;
        padding: 20px;
        border-radius: 8px;
        box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
    }
    
    /* These classes will override some Salesforce styling */
    .slds-scope .slds-button {
        background-color: #0070d2;
        color: white;
    }
</style>
```

## Troubleshooting

### Common Issues and Solutions

1. **CORS Errors**: Ensure the domain hosting your HTML page is added to the CORS Allowed Origins list in Salesforce Setup.

2. **Authentication Issues**: Make sure your Experience Cloud site is properly configured for guest user access if you're embedding flows for unauthenticated users.

3. **Component Not Loading**: Verify that your Lightning app and component are properly configured with the `access="GLOBAL"` attribute.

4. **Flow Not Starting**: Double-check that your flow API name is correct in the component controller.

5. **Lightning Out Script Not Loading**: Ensure the path to your Experience Cloud site's Lightning Out JavaScript file is correct.

### Additional Resources

- [Salesforce Lightning Out Documentation](https://developer.salesforce.com/docs/atlas.en-us.lightning.meta/lightning/lightning_out.htm)
- [Salesforce Flow Documentation](https://help.salesforce.com/articleView?id=flow.htm)
- [Experience Cloud Sites Documentation](https://help.salesforce.com/articleView?id=networks_overview.htm)

---

For more information, refer to the detailed tutorial: [How to Place Screen Flows on Your Website](https://medium.com/@justindixon91/hackforce-how-to-place-screen-flows-on-your-website-4d93665f0c9c)