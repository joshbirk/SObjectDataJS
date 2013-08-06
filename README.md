<h1>SObjectData for the Salesforce Mobile SDK</h1>
SObjectData is a JavaScript library to abstract low-level access to the data and CRUD functionality of the Salesforce Mobile Pack.  In particular, it allows developers to utilize SmartSync without the need to use Backbone (or a Backbone-like framework), instead only requiring:

* underscore.js
* force.entity.js 
* either ForceTK (for API access) or RemoteTK (for Javascript Remoting access)	

As such, it is suitable for use with non-MVC libraries like jQuery Mobile, or in conjunction with Salesforce Mobile Design Templates.  For using libraries like Angular or Knockout - see their respective mobile packs.

<h2>Instantiating</h2>
SObjectData can be instantiated in a number of ways, and it is best to think of each instance of SObjectData has a handler for a unique set of data.  So if you are going to track both Account and Contact information in the same application, you would use two instances of SObjectData.

The constructor for an SObjectData instance:

```
function SObjectData(stype,fields) {
	this.dataArray = new Array();
	this.dataObject = null;
	this.cacheConfig = {
		cache : null,
		cacheMode : Force.CACHE_MODE.SERVER_ONLY
	}
	
	this.SObjectType = stype;
	this.fields = fields;
	
}
```

You can leave some of the incoming parameters blank/null:
```
var Contacts = new SObjectData();
var Contacts = new SObjectData('Contact');
var Contacts = new SObjectData('Contact',['Id','Name','Phone']);
```

In fact, if you are going to fill the data with a query - there's little need to set the SObjectType or Fields ... *fetch* will do that for you.  The fields array will rarely need to handled via the constructor, as SObjectData will also track the fields based on the current operation.

<h2>Querying</h2>
To get data, SObjectData has a *fetch* method.  This method has three params, all required: type, query and callback.  
```
Contacts.fetch('soql',"SELECT id, firstName, lastName, phone from Contact WHERE phone != '' LIMIT 100",function() {
    showContacts(Contacts.data());
});
```

*fetch* will automagically update the SObjectType and Fields based on the data being returned from the server.

<h2>Handling Records</h2>
SObjectData has a few methods for conveniently accessing data currently stored within an instance.  The most common will be *data*:
```
Contacts.data()
```

Which simply returns the current array.  If you wish to manually fill the array:
```
Contacts.data([record JSON,record JSON])
```

Will replace the current array and update the type and fields as needed.  There are also two methods for searching within the array:
```
var contact = Contacts.findRecordById(record ID);
var contact = Contacts.findRecordByIndex(record Index);
```

You can also get the first (or manually set) record in the array directly as an object:
```
var contact = Contacts.record();
```

<h1>Creating and Updating Data</h1>
SObjectData has a *create* method which aims to be ridiculously versatile.  It can be used to create blank records:
```
//get blank record based on current type and fields
var contact = Contacts.create(); 

//get blank record based on new fields
var contact = Contacts.create(['Id','FirstName','LastName']);
```

Or it can be used to create a new data point:
```
record = Contact.create('Contact',{
    'FirstName' : 'Joe',
    'LastName' : 'Test,
    'Phone' : '555-555-5555',
});
```

When creating a new data point, the data will be added to the array (or create an array if it is null), and set the current record (*record()*) to be the new record.  A developer may also simply update existing records in the data array:

```
var record = Contacts.findRecordById(contact Id);
record.FirstName = 'Joe';
```

<h1>Syncing Data</h1>
To insert or update records back to the Salesforce Platform (and/or cache), SObjectData uses the *sync* method.  The sync method requires a pointer to a record and a callback.  If no pointer is provided, SObjectData assumes the record in question is the one currently held by record();  If the data has an Id field, it will update the record - otherwise it will insert. 

```
//insert or update the current record()
Contacts.sync(function() {
    getAllContacts();
    });

//insert a new record - note this requires the SObjectType to be set on the SObjectData instance
Contacts.sync({
    'FirstName' : 'Joe',
    'LastName' : 'Test,
    'Phone' : '555-555-5555',
}, function() {
    getAllContacts();
    });

//update a record by id - note this requires the SObjectType to be set on the SObjectData instance
Contacts.sync(Contacts.findRecordById(record ID), function() {
    getAllContacts();
    });

//update a record by index in data() - note this requires the SObjectType to be set on the SObjectData instance
Contacts.sync(0, function() {
    getAllContacts();
    });
```

<h1>Deleting Data</h1>
Deleting data is done the same way as *sync*, except with the method *remove* which forces the record to be removed instead of inserted or updated.
```
//delete the current record()
Contacts.remove(function() {
    getAllContacts();
    });
````

<h1>Error Handling</h1>
Each instance of SObjectData has error handler which will be called whenever *sync* (or *remove*) returns an error state.  By default, this error handler is:
```
SObjectData.prototype.errorHandler = function(error) {
	console.log(error);
}
```

If a developer wants to handle errors differently, this method can be overridden.


