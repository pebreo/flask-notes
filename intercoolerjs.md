
ic-get-from

ic-post-to

<ic-post-to="/foo" ic-trigger-on="onload">

// run when these ic events fire
$(document).bind('success.ic', function(){
    console.log('ic fired'); 
});

// use in place of jQuery
// use this to add logic in your ic generated html 
Intercooler.ready(function(elt){
   console.log('intercooler is ready'); 

});

ic-global-include

ic-include


ic-target

ic-deps 