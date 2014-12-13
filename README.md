mongoose-autorefs
=================
Mongoose plugin for automatic updating of referenced documents

**Usage**

Add `autoref` as a plugin to a schema, passing in an options array. For example:

```
var autoref = require('mongoose-autorefs');
var mongoose = require('mongoose');

var companySchema = new mongoose.Schema({
    _id: ObjectId,
    name: String,
    employees: [{type: ObjectId, ref: 'Person'}],
    interviewees: [{ type: ObjectId, ref: 'Person' }]
});
companySchema.plugin(autoref, [
    'employees.employer',
    'interviewees.interviewers'
]);

var personSchema = new mongoose.Schema({
    _id: ObjectId,
    name: String,
    partner: { type: ObjectId, ref: 'Person' }, // 1-1 self
    friends: [{ type: ObjectId, ref: 'Person' }], // *-* self
    employer: { type: ObjectId, ref: 'Company' }, // 1-*
    interviewers: [{ type: ObjectId, ref: 'Company' }] // *-*
});
personSchema.plugin(autoref, [
    'partner.partner',
    'friends.friends',
    'employer.employees',
    'interviewers.interviewees'
]);
```


**The `options` array**

This array defines one or more paths to referenced documents to update on a `save`.

In the above example, when saving a `Company` document with an `employee`, autoref will automatically update the referenced `Person` document, setting it's `employer` field to the `_id` of the saved `Company`.

This will also work for arrays and arbitraryily nested documents.


**IMPORTANT NOTES**

**1.**  The autoref behaviour is implemented using mongoose's post save middleware. This means the autoref functionality **will only execute on a `save`**. The various `findAndModify` methods **do not** fire the post save middleware.

**2.**  Since the post save middleware currently does not include a `next` or `done` callback, you can only determine when autoref has completed by listening to the associated event on the document's `Model`.
You should use the `completeEvent(id)` method to get the name of the event you want to listen to. The event passes the updated document.
So for example, saving a `Person` document and listening for the complete event:

```
var autoref = require('mongoose-autorefs');

var person = new Person({name: 'Mike'});
person.save(function(err, mike){
    // Listen for the autoref completed event
    Person.on(autoref.completeEvent(mike._id), function(mike){
        // Do some stuff
    });
});
```
=======