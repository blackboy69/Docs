# Task Based API for OneLegal Filing Workflow

## Background
Proposal for a task based filing API. OneLegal's "secret sauce" is not the technology, it is the easy to use workflow that every customer uses on a daily basis. 

This workflow currently is encapsulated in the UI layer and very difficult to expose as an API to allow integrations. 

Secondly, any UI changes require deep changes to the UI forms. Since the UI and task flow are tied to gether, changes are difficult.

	Changing the workflow will always require lots of work. The workflow is the onelegal competetitive advantage!


## Proposal
Seperate the workflow from the UI by using  a Task based REST API.

The OneLegal filing API preserves state based on previous choices. It usees the  [CQRS](https://bjoernkw.com/2017/04/02/design-patterns-event-sourcing-and-command-and-query-responsibility-segregation-cqrs/) design pattern, but does not attach state to the requests via cookies or client object.

The API is dynamic. Some endpoints cannot be determined beforehand, endpoints may be dynamically presented **based on the data**. 

This type of API will fulfill the goals of enabling next generation UI development, 3rd party API integration and modernization of the platform.


## Guiding Principals
API will be REST based and adhere to stateless design.  The state of the document filing order is determined via URL per REST best practicies.

-  Successfull submission of document(s) will require multiple api calls with dependent data from previous calls.
 - API method calls can be modeled with a dependency graph. 
 - API calls shall return a list of `links` when user choice is needed. 
	- for example: `state =>court => court location => case type`
 - API calls may return a `form` when user input is needed.
	- A form means the Api needs to accept post data.
	- form elements include all fields needed for the next step
	- The form element describe how to validate and submit for the next step.


## Sample API based workflow

The request/responses illustrate the logic behind a theoretical submission. 

`GET /Filing`

**Response 200 OK**
```
{
  "id": 1,
  "links": [
    {
      "title": "Case Initiation",
      "href": "/Filing/Initiate"
    },
    {
      "title": "Subsequent Filing",
      "href": "/Filing/Subsequent"
    }
  ]
}
```

`GET /Filing/Initiate`

**Response 200 OK**
```
{
    id:213
   "links": [ 
   {
	"title": "State",
	"href": "/Filing/Initiate/State"
   },
   {
	"title": "Federal",
	"href": "/Filing/Initiate/Federal"
   }	   
  ]
}
```
   
   
`GET /Filing/Initiate/State`

**Response 200 OK**
```
{
   "links": [ 
   {
	"title": "California",
	"href": "/Filing/Initiate/State/California"
   },
   {
	"title": "New York",
	"href": "/Filing/Initiate/State/NewYork"
   }	
  ]
}
```

`GET /Filing/Initiate/State/California`

**Response 200 OK**
```
{
   "links": [
   {
		"title": "Sacramento County, Superior Court of California",
		meta: [{name: "Location", value: "Sacramento, 301 Bicentennial Circle (Carol Miller)", "Submission Deadline" :"3pm",  "Court Info":{["name","address","telephone"]}}],
		"href": "/Filing/Initiate/State/California/6122"
   },
   {
		"title": "Sacramento County, Superior Court of California",
		meta: [{name: "Location", value: "3341 Power Inn Road (Family Law-Probate)", "Submission Deadline" :"3pm", "Court Info":{["name","address","telephone"]}}],
		"href": "/Filing/Initiate/State/California/691"
    },
    {
		"title": "Sacramento County, Superior Court of California",
		meta: [{"Location": "Sacramento, 720 9th Street (Sacramento)", "Submission Deadline" :"3pm", "Court Info":{["name","address","telephone"]}}],
		"href": "/Filing/Initiate/State/California/8342"
    },
  ]
}
```

`GET /Filing/Initiate/State/California/6122`

**Response 200 OK**
```
{
   id: 82
   "links": [
   {
		"title": "Civil Limited",
		"href": "/Filing/Initiate/State/California/6122/CivilLimited"
   },
   "links": [
   {
		"title": "Probate/Mental Health",
		"href": "/Filing/Initiate/State/California/6122/ProbateMentalHealth"
   },
   ]
}
```

   
`GET /Filing/Initiate/State/California/6122/CivilLimited`

**Response 200 OK**
```
{
	id: 341
	"links": [
	{
		"title": "Collections",
		"href": "/Filing/Initiate/State/California/6122/CivilLimited/Collections",
	},
	"links": [
	{
		"title": "Civil Rights,
		"href": "/Filing/Initiate/State/California/6122/CivilLimited/CivilRights
	},
	]
}
```


The method below contains all of the policy information reduced down to one call. The field are based on the court, case type, etc from the URL.
		
`GET /Filing/Initiate/State/California/6122/CivilLimited/Collections`

**Response 200 OK**
```
{		
   id: 82
   form:  // FORM Means you must POST json "fields" to "post"
   {
		id: 42708,
		"title": "Collections",
		post: "/Filing/Initiate/State/California/6122/CivilLimited/Collections",		
		fields: [{ 			
		            	name: "Plantiff", 
				field: "plantiff",
				required: true,
				type: text
			 }, 
			 {  
			    	name: "Attorney",
				field: "attorneyId"
				required: true,
				type: id
			 },
			{
				name: "Representing", 
				required: true				 
				values: ["Petitioner","Cross Defendant","Plaintiff","Creditor"],
				type: text
			},
			{ 		
				name: "Defendant", 
				field: "defendant",
				required: false,
				type: text
			}, 
			{ 						
				name: "Defendant", 
				field: "defendant",
				required: false,
				type: text
			}, 
			{ 						
				name: "Client Billing Code", 
				field: "clientBilingCode",
				required: false,
				type: text
			}, 		
			{ 						
				name: "Email Completion Noticies To", 
				field: "completionNoticeEmail",
				required: false,
				values: ["None","Contact"],
				type: text
			},
			{
				name: "Service Mode",
				field "serviceMode",
				required: true,
				values: "eFiling Only","eFiling with Physical Process Serving"
			}
		]
   }
}   
```


The following call creates the Order and returns its ID. That id can be used to attach multiple documents later.


`POST /Filing/Initiate/State/California/6122/CivilLimited/Collections`

**Request HTTP 1.1/POST**
```
{
   Plantiff: "Recovery Collections",
   Representing: "Petitioner",
   Defendant: "John C. Smith",
   BillingCode: "41bv15a"
   ContactId: 123,
   AttorneyId: 321,
   completionNoticeEmail: "None",
   clientBilingCode: "23"
   serviceMode:  "eFiling Only"
}
```

**Response 200 OK**
```
{
	id: 341,
	status: "draft",
	"links": [
	{
		"title": "Upload Document",
		"href": "/Document/341",
	},
}
```


`GET /Document/341`

**Response 200 OK**
{
	id: 423	
	"title": "Attach Document
    form:  
    {
		id: 8108,
		post: "/Documents/341/Attach",		
		fields: [{ 						
		            name: "Document Type", 
					field: "documentType",
					required: true,
					values: ["Appellate Division Petition for Writ (APP-151)", "Application for Deposition Subpoena in Action Outside California"],
					type: text
				  },
				  {
					name: "Document Data",
					field: "documentBytes",
					required: true,					
					type: "application/pdf",					
				  }
		]				  
}
   
`POST /Documents/341/Attach`

**Response 200 OK**
```
...
```

`POST /SubmitOrder`

**Request HTTP 1.1/POST**
```
{
	orderId: 341
}
```

**Response 200 OK**
```
{Success: true}
```


## Further Reading
- [CQRS and REST: the perfect match](https://lostechies.com/jimmybogard/2016/06/01/cqrs-and-rest-the-perfect-match/)
- [Task based UI](https://cqrs.wordpress.com/documents/task-based-ui/)
- [Event Sourcing and CQRS](https://bjoernkw.com/2017/04/02/design-patterns-event-sourcing-and-command-and-query-responsibility-segregation-cqrs/)
- [Stackoverflow- REST + TASK UI ](https://stackoverflow.com/questions/7147340/task-oriented-user-interface-and-rest-application-server-api)


