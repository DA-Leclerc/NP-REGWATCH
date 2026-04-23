---
id: microsoft-copilot-data-residency
url: https://learn.microsoft.com/en-us/microsoft-365/enterprise/m365-dr-service-copilot
language: en
fetched_at: 2026-04-23T06:00:00-04:00
source_category: vendor-policy
priority: critical
---

Skip to main content  Skip to Ask Learn chat experience 

This browser is no longer supported.

Upgrade to Microsoft Edge to take advantage of the latest features, security updates, and technical support. 

[ Download Microsoft Edge ](https://go.microsoft.com/fwlink/p/?LinkID=2092881 ) [ More info about Internet Explorer and Microsoft Edge ](https://learn.microsoft.com/en-us/lifecycle/faq/internet-explorer-microsoft-edge)

Table of contents  Exit editor mode

Ask Learn Ask Learn

Reading mode Table of contents Read in English Add Add to plan [ Edit ](https://github.com/MicrosoftDocs/microsoft-365-docs/blob/public/microsoft-365/enterprise/m365-dr-service-copilot.md)

* * *

#### Share via

Facebook x.com LinkedIn Email

* * *

Copy Markdown Print

* * *

Note

Access to this page requires authorization. You can try signing in or changing directories. 

Access to this page requires authorization. You can try changing directories. 

# Data Residency for Microsoft 365 Copilot and Copilot Chat

Feedback

Summarize this article for me 

##  In this article 

## Overview

Service documentation: [Microsoft 365 Copilot overview](/en-us/microsoft-365-copilot/microsoft-365-copilot-overview) and [Data, Privacy, and Security for Microsoft 365 Copilot](/en-us/microsoft-365-copilot/microsoft-365-copilot-privacy)

Capability Summary: Microsoft 365 Copilot is an AI-powered productivity tool that coordinates large language models (LLMs), content in Microsoft Graph, and the Microsoft 365 apps that you use every day, such as Word, Excel, PowerPoint, Outlook, and Teams. This integration provides real-time intelligent assistance, enabling users to enhance their creativity, productivity, and skills. The following applications provide the ability to interact with Microsoft 365 Copilot: Microsoft Word, Excel, PowerPoint, Loop, Outlook, Teams (Chat, Meetings, Calls, Whiteboard), and OneNote.

The content of interactions and the related semantic index with Microsoft 365 Copilot are stored at rest in the relevant _Local Region Geography_.

[Microsoft 365 Copilot Chat](/en-us/copilot/overview) (formerly Microsoft Copilot for Entra account users) offers secure, AI chat that adds pay-as-you-go agents.

## Data Residency Commitments Available for Microsoft 365 Copilot and Copilot Chat

### Product Terms

Required Conditions:

  1. _Tenant_ has a sign-up country/region included in Australia, Brazil, Canada, the European Union, France, Germany, India, Japan, Norway, Qatar, South Africa, South Korea, Sweden, Switzerland, the United Kingdom, the United Arab Emirates, or the United States.



**Commitment:**

_For current language, refer to the[Privacy and Security Product Terms](https://www.microsoft.com/licensing/terms/product/PrivacyandSecurityTerms/all) and view the section titled "Location of Customer Data at Rest for Core Online Services."_

### Advanced Data Residency (ADR) add-on

Required Conditions:

  1. _Tenant_ has a sign-up country/region included in _Local Region Geography_.
  2. _Tenant_ has a valid Advanced Data Residency subscription for all users in the _Tenant_
  3. For existing _Tenant_ that has data stored in a _Macro Region Geography_ , the _Tenant_ Global Admin must opt in to move the _Tenant_ data into the _Local Region Geography_.
  4. The Microsoft 365 Copilot subscription customer data is provisioned in _Local Region Geography_.



Important

Microsoft recommends that you use roles with the fewest permissions. This helps improve security for your organization. Global Administrator is a highly privileged role that should be limited to emergency scenarios when you can't use an existing role.

**Commitment:**

Refer to the [ADR Commitment page](m365-dr-commitments?view=o365-worldwide#microsoft-365-copilot-and-copilot-chat) to understand the specific data at rest commitments for Microsoft 365 Copilot. Examples of the committed data include:

  * "Content of Interactions" such as the user's prompt and the response from Microsoft 365 Copilot or Microsoft 365 Copilot Chat, including citations to any information used to ground Microsoft 365 Copilot's response.



### Multi-Geo add-on

Required Conditions:

  1. _Tenants_ have a valid Multi-Geo subscription that covers all users assigned to a _Satellite Geography_
  2. Customer must have an active Enterprise or CSP Partner Agreement.
  3. Total purchased Multi-Geo units must be greater than 5% of the total eligible licenses in the _Tenant_.



**Commitment:** Multi-Geo capabilities in Microsoft 365 Copilot and Microsoft 365 Copilot Chat enable content of interactions with Microsoft 365 Copilot and Microsoft 365 Copilot Chat to be stored at rest in a specified _Macro Region Geography_ or _Local Region Geography_ location. Both Microsoft 365 Copilot and Microsoft 365 Copilot Chat use the Preferred Data Location (PDL) for users and groups to determine where to store data. If the PDL isn't set or is invalid, data is stored in the _Tenant's Primary Provisioned Geography_ location. The _Geography_ where the content of interactions with Microsoft 365 Copilot and Microsoft 365 Copilot Chat are stored is determined by the PDL of the user interacting with Microsoft 365 Copilot or Microsoft 365 Copilot Chat, respectively. This means that the storage of content of interactions for users in different regions will be based on their respective PDL configurations.

To find the current location of a user's content of interactions with Microsoft 365 Copilot by referencing the PDL configuration for that user. Refer to [Multi-Geo Testing](m365-multi-geo-user-testing?view=o365-worldwide).

**Illustrative examples**

**Collaboration Experience** Two people are working together on a Microsoft Word document. User A authored the document and stored it in the OneDrive for Business personal storage site, which is located in France. User B is in Canada and asks Microsoft 365 Copilot to rewrite a paragraph in the document. The paragraph User B submitted as the prompt, as well as the rewrite options Microsoft 365 Copilot provides (the "content of interactions" in this case) are stored in Canada; the original document remains in France, as does any rewrite the user accepts into that document.

**Teams Meeting Experience** Microsoft Teams meeting recording video location is determined by the user PDL that starts the recording, or when meetings have an automatic recording policy, the location is determined by the meeting organizer. When users in other regions interact with Microsoft 365 Copilot in Teams, those user prompts and corresponding responses are stored in the location of the user that asks the Microsoft 365 Copilot questions.

### Migration and User Experience

When a user interacts with Microsoft 365 Copilot (using apps such as Word, PowerPoint, Excel, OneNote, Loop, or Whiteboard), we store data about these interactions. The stored data includes the user's prompt and Copilot's response, including citations to any information used to ground Copilot's response. We refer to the user's prompt and Copilot's response to that prompt as the "content of interactions" and the record of those interactions is the user's Copilot interaction history. For example, this stored data provides users with Copilot interaction history in [Microsoft 365 Copilot Chat](https://support.microsoft.com/topic/get-started-with-copilot-for-microsoft-365-5b00a52d-7296-48ee-b938-b95b7209f737) and [meetings in Microsoft Teams](https://support.microsoft.com/office/get-started-with-copilot-in-microsoft-teams-meetings-0bf9dd3c-96f7-44e2-8bb8-790bedf066b1). This data is processed and stored in alignment with contractual commitments with your organization's other content in Microsoft 365, such as [Advanced data residency in Microsoft 365](advanced-data-residency?view=o365-worldwide).

When a customer elects [Advanced data residency in Microsoft 365](advanced-data-residency?view=o365-worldwide), they are subject to [ADR Migration](advanced-data-residency?view=o365-worldwide#data-migration-management). For detailed information regarding customer impact during the migration, please refer to [Data Residency for Microsoft Teams](m365-dr-service-teams?view=o365-worldwide#user-experience).

### How can I determine customer data location?

You can find the actual data location in Microsoft 365 admin center. In the coming months, you will be able to find the actual data location for committed data, by navigating to **Settings > Org settings > Organization profile > Data location**.

* * *

## Feedback

Was this page helpful? 

Yes No No

Need help with this topic? 

Want to try using Ask Learn to clarify or guide you through this topic? 

Ask Learn Ask Learn

Suggest a fix? 

* * *

##  Additional resources 

* * *

  * Last updated on  2026-02-19 



### In this article

Was this page helpful?

Yes No No

Need help with this topic? 

Want to try using Ask Learn to clarify or guide you through this topic? 

Ask Learn Ask Learn

Suggest a fix? 

en-us

[ Your Privacy Choices](https://aka.ms/yourcaliforniaprivacychoices)

Theme

  * Light 
  * Dark 
  * High contrast 



  *   * [AI Disclaimer](https://learn.microsoft.com/en-us/principles-for-ai-generated-content)
  * [Previous Versions](https://learn.microsoft.com/en-us/previous-versions/)
  * [Blog](https://techcommunity.microsoft.com/t5/microsoft-learn-blog/bg-p/MicrosoftLearnBlog)
  * [Contribute](https://learn.microsoft.com/en-us/contribute)
  * [Privacy](https://go.microsoft.com/fwlink/?LinkId=521839)
  * [Consumer Health Privacy](https://go.microsoft.com/fwlink/?linkid=2259814)
  * [Terms of Use](https://learn.microsoft.com/en-us/legal/termsofuse)
  * [Trademarks](https://www.microsoft.com/legal/intellectualproperty/Trademarks/)
  * (C) Microsoft 2026


  *[en]: English
