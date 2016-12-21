---
layout: post
title:  "Sticky headers and JQuery trick to sync tables widths"
date:   2016-12-21 16:48:10 -0500
---


I was working on a website that has a large table of software data in it that, when long enough, requires one to scroll to see the table contents. What I wanted to do was make a "sticky" header that docked to the top of the page that duplicated the header in the main table. 

The web page is built using PHP so there are several components that are involved. One is the page header which draws out the logo and menu buttons. Part of this header is also sticky so when I scroll, both the menu bar and the table header dock at the top. 

Here is the main sticky menu bar which contains a shopping cart button which I want people to have access to:

```
<div class="main-nav">
      <div class="container">
      <div class="row headSocial"
    <div class="col-lg-12">  
      <div class="col-md-3">
        <ul>
         <li><a class="customer_menu" href="index.php">Dashboard <span class="glyphicon glyphicon-dashboard" aria-hidden="true"></span></a></li>
        </ul>
      </div>
      <div class="col-md-3"></div>
      <div class="col-md-6">
        <ul class="navbar-right moveLeft">
           <li> <a id="headCart" class="btn btn-success btn-md" data-trigger="hover"  onclick="checkCart()" hidden="hidden">&nbsp<span class="glyphicon glyphicon-shopping-cart" aria-hidden="true">Cart: <span id="headCartAmt" class="badge" :empty></span></span></a></li>
            <li> 
             <a id="logout" style="display: none;" class="customer_menu_sm" href="index.php?logout">
                  Log Out <span class="glyphicon glyphicon-log-out" aria-hidden="true"></span> </a>
            </li>
          </ul>
      </div>
    </div>
```
	
	
To accomplish this, I duplicated the table header of the main table in the header PHP document and set the header to display none. The containing div is called 'tempDiv' and uses a class of 'main-nav2'.  Note the added col# classes which will be used later. (the bld clas is just to make the text bold face). 


```
<div id="tempDiv" class="main-nav2" style="display:none;">
   <table id="tempTable" class="table">
   <thead>
   <tr >
    <th class="bld col1">Registration Number</th>
    <th class="bld col2">Name</th>
    <th class="bld col3">Maintenance Renewal Due <a href="#" data-toggle="popover" title="Info on Maintenance" data-placement="top" data-content="This is the date your maintenance contract expires for this license." data-original-title="More Info"><i class="glyphicon glyphicon-info-sign orange"></i></a></th>
    <th class="bld col4">License Type</th>
    <th class="bld col5">Single/ Network</th>
    <th class="bld col6">Software</th>
    <th class="bld col7">Network Seats</th>
    <!--  //echo '<th class="bld"># of Unlocks<a href="#" data-toggle="popover" title="Info on Unlocks" data-placement="top" data-content="A count of how many times a license has been unlocked. If unlocking is abused the license may be ineligible for immediate unlocking." data-original-title="More Info"><i class="glyphicon glyphicon-info-sign orange"></i></a></th>  -->
    <th class="centerTxt bld col8" colspan="3">Software Options</th>
  </tr>
  </thead></table>
</div>
```


To get the parts to stick, I use this css:


```
.main-nav {
    background: #0c4884  !important;
    padding: 3px 0;
    height:65px;
    text-align: center;
    z-index: 999;
}

.main-nav a {
   z-index: 999;
    text-decoration:none;
    display:inline-block;
    padding:15px 19px;
}

.main-nav.stickytop {
    position:fixed;
    top:     0;
    width:   100%; 
    height:  65px;
    z-index: 999;
}

.main-nav2.stickytop {
    position:fixed;
    top:     0px;
    height:  57px;
    width:   100%; 
    z-index: 9999;
}
.main-nav2 {
    background: #fff ;
    height:     57px;
    margin-left:15px;
    margin-top: 65px;
    vertical-align: baseline;
    text-align: center;
    z-index:    -1;
}
```


But the css isn't enough to make it work, we need a some JQuery. Here's the code and an explaination:


```
$(window).scroll(function() {  //triggers when the page is scrolled
    var amount = $('#tempHead').length!=0 ? 167 : 67;  //See explanation below
    if ($(this).scrollTop() >= amount) {
        $('.main-nav').addClass('stickytop');  //add class to the menu div to make it stick
        
        if($('#tempHead').length!=0 ) {  //if the page has the main table
        $('.main-nav2').show();  //unhide the table header clone
        $('.main-nav2').addClass('stickytop')  //and make it stick to the top
        initialHeadAdjust();  //**this is the tricky part
        }
    }
    else {   //this resets the divs back to normal without the stickytop class
        $('.main-nav').removeClass('stickytop');  
        $('.main-nav2').removeClass('stickytop');
        $('.main-nav2').hide();

        if($('#tempHead').length!=0 ) {  //to be sure that the original main table header is not hidden
          $('#tempHead').show(); 
        }
    }
});
```

The main table of the page has the id 'tempHead'. Since the page header is used for all pages and not all pages have the main software table in it, I don't want the same action to happen on all pages, just the one where the main table is. This is why there are the if statements looking for ``` if($('#tempHead').length!=0 )``` which would be true if the id is present. 
So the first var assignment sets the scroll amount to 167px if the table exists and only 67 on other pages. 
Then we examine how much the page has scrolled. If the amount is exceeded, we will add the class 'sticktop' to the div with the menu header in it. The css for the sticktop fixes the position and sets the z-index higher to be on top of the scrolling content. 

This all worked fine, but even with using the same css info linked to my 'table' class in both the main table header and the cloned one, the widths of the table cells were not matching up. This is there the ``` initialHeadAdjust();``` function comes in. In my main jacascript file for the project, I have two ways to keep the sizes synced. They do the same thing just one is called once and the other is called during window resize. 

The main table id is 'tempHead', the cloned header id is 'tempTable'.

Here's the code:

```
//matches the floating header widths to match the table below
function initialHeadAdjust(){
    for(var i=1; i<9; i++) {  //I could use a for each here but I have a set number of cells
            var newWidth = $('#tempHead th.bld.col'+i)[0].clientWidth;  //get the width of the header in the main table
            $('#tempTable th.bld.col'+i).eq(0).css('width', (newWidth) + "px");  //apply the width to the cloned header
        }
}

//now every time the windw is resized and the table cells change, the two will sync and line up
$(window).on('resize', function(){  
    for(var i=1; i<9; i++) {
            var newWidth = $('#tempHead th.bld.col'+i)[0].clientWidth;
            $('#tempTable th.bld.col'+i).eq(0).css('width', (newWidth) + "px");
        }
});
```

So here is an example of what it looks like in action: [Example Video](http://recordit.co/nkRUGTKee1)

I hope you are able to get something from this that's useful. Let me know if you have any questions on the code or how it works!

Happy Holidays!
