Meteor DHTMLX Scheduler with Live Update
====================================

Updates automatically to all connected clients (autopublish and insecure packages removed)



####Include

  - imgs/ & imgs_dhx_terrace/ in /app/public/
  - dhtmlxscheduler.css in /app/client/stylesheets/
  - dhtmlxscheduler.js in /app/client/lib/
  

####Models.js (/app/models.js)


```sh
Meteor.events = new Meteor.Collection('events');
```

####Client.js (/app/client/client.js)

```sh
var  initScheduler = function () {

			$(".myscheduler").dhx_scheduler({
                xml_date:"%Y-%m-%d %H:%i",
                prevent_cache:true,
                date:new Date(),
                mode:"month"
            });

			scheduler.attachEvent("onEventAdded", function(id,ev){
			 	Meteor.call("insert", ev, function(error, event_id) {
				});	
			});
			scheduler.attachEvent("onEventChanged", function(id,ev){
				Meteor.call("update", ev, function(error, event_id) {
				});
			});
			scheduler.attachEvent("onEventDeleted", function(id){
				Meteor.call("delete", id, function(error, event_id) {
				});	
			});
};
Template.scheduler.rendered = function(){
 
    initScheduler();
    Meteor.subscribe("jobs");
    Meteor.autorun(function(){
		var evs = Meteor.events.find().fetch();
		var load = [];
		$.each( evs, function(i, e){
		load[i] = {id:e._id,start_date:e.start_date,end_date:e.end_date,text:e.text} ;
		});
		scheduler.clearAll();
		scheduler.parse(load, 'json');
    });
      
};

```

####Server.js (/app/server/server.js)

```sh
Meteor.publish("jobs", function () {

    return Meteor.events.find({});
  
});

Meteor.methods({
        'insert': function(ev) {
            var event_id = Meteor.events.insert(ev);
            return event_id;
        },
		'update': function(ev) {
            var event_id = Meteor.events.update(ev.id, ev);
            return event_id;
        },
		'delete': function(id) {
            var event_id = Meteor.events.remove(id);
            return event_id;
        }
});
```

####Scheduler.html (/app/scheduler.html)

```sh
<head>
  <title>scheduler</title>

<style type="text/css" media="screen">
    html, body{
        margin:0px;
        padding:0px;
        height:100%;
        overflow:hidden;
    }
</style>
</head>


<body>
  {{> scheduler}}
</body>


<template name="scheduler">

<div class="myscheduler" style='width:100%; height:100%;'></div>

 
</template>
```



