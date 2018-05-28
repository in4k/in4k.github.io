
function verify(url, text){
		if (text=='')
			text='Are you sure you want to delete this comment?';
		if (confirm(text)){
			document.location = url;
		}
		return void(0);
	}
// setup everything when document is ready
jQuery(document).ready(function($) {

  			if (WPWallSettings.expand_box != 'on')
	   		$("#wall_post").css("display","none");
	
			  $("#wall_post_toggle").click(function(){
			  
			  		
			       $("#wall_post").slideToggle("fast");
			       
			       $("#wall_post #comment").focus();
			  		 $("#wall_post #author").focus();
			       
			     
			  });
			  
        $('#wallform').ajaxForm({ 
          // target identifies the element(s) to update with the server response 
        target: '#wallcomments', 
       
        // handler function for success event
        success: function(responseText, statusText) {            
            
            $('#wallresponse').html('<span class="wall-success">'+'Thank you for your comment!'+'</span>');               
            $('#submit_wall_post').attr('value', 'Submit');
            $("#wall_post").hide("fast");
            
            $("#wall_post #author").attr('value', '');
            $("#wall_post #comment").attr('value', '');
            $("#wall_post #wpwall_comment").attr('value', '');
           
        } ,
        
        // handler function for errors
        error: function(request) {                        
            
            // parse the response for WordPress error
            if (request.responseText.search(/<title>WordPress &rsaquo; Error<\/title>/) != -1) {
            	
							var data = request.responseText.match(/<p>(.*)<\/p>/);
							$('#wallresponse').html('<span class="wall-error">'+ data[1] +'</span>');
					} else {
							
							$('#wallresponse').html('<span class="wall-error">An error occurred, please notify the administrator.</span>');
					}       
					
					  $('#submit_wall_post').attr('value', 'Submit');       
					  
					   $("#wall_post #author").attr('value', '');
             $("#wall_post #comment").attr('value', '');                      
        } ,
        beforeSubmit: function(formData, jqForm, options) { 
        	
        	// clear response div
        	$('#wallresponse').empty();  
        
          for (var i=0; i < formData.length; i++) { 
              if (!formData[i].value) {
                  $('#wallresponse').html('<span class="wall-error">'+'Please fill in the required fields.'+'</span');                                    
                  return false; 
              } 
              
          } 
       
          $('#submit_wall_post').attr('value', 'Please wait');                              
        }              
    });   
    
    
     $('.wallnav #img_left').click(function(){
     
     		var page= $('#wallcomments #page_left');
     		var wallform = $('#wallform');
	  		
	  		if (wallform[0])		  				  				 		  				  				 
	     		$('#wallcomments').load(wallform[0].action+'?refresh=' + page[0].value );
      });
     
     
     $('.wallnav #img_right').click(function(){
     
     		var page= $('#wallcomments #page_right');
     		var wallform = $('#wallform');
	  		
	  		if (wallform[0])		  				  				 
	     		$('#wallcomments').load(wallform[0].action+'?refresh=' + page[0].value );
      });
     
     refreshtime=parseInt( WPWallSettings.refreshtime);
     if (refreshtime) 
	     timeoutID = setInterval(refresh, (refreshtime  < 5000 ) ? 5000: refreshtime );
	     
	     
	   function refresh() {
	     	
	     			var wallform = $('#wallform');
	     			var page= $('#wallcomments #wallpage');
	     			
	  				if (wallform[0])		  				  				   				  				 
	     	    	$('#wallcomments').load(wallform[0].action+'?refresh=' +  page[0].value);
	          
	   }     
});