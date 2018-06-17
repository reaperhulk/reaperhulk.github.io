---
author: admin
comments: true
date: 2010-03-04 20:34:09+00:00
layout: post
slug: jqgrid-local-data-live-search
title: jqGrid Local Data Live Search
wordpress\_id: 986
tags:
- javascript
- jqgrid
- jquery
---

jqGrid is an incredibly powerful and flexible plugin for jQuery that allows you to build data grids using nothing but Javascript, HTML, and CSS.  I recently wanted to allow live filtering of local results (no AJAX queries, just parsing local data) based on a search string.  [View the demo](/assets/media/other/jqgrid_live_search/) and then follow along below.



### Basic HTML Structure


Below is some very basic HTML that you can use to build a jqGrid.

```html






## jqGrid Live Search Demo








```




### Create A Grid


This snippet will create a grid using the "smoothness" jQuery-UI theme.  This grid has 3 columns and obtains its data in JSON format from "data.php".  Notice the global declaration of searchColumn and the call to fetch an array of all the data from the name column when the gridComplete event fires.  You could grab the other columns as well, but for this example we're only going to search on name.

```javascript

var gridimgpath = 'css/smoothness/images';
var grid = jQuery('#list');
var searchColumn;

grid.jqGrid({
	url:'data.php',
	datatype: 'json',
	mtype: 'POST',
	colNames: ['Name','Data','Date'],
	colModel :[
	  {name:'name', index:'name', width:140},
	  {name:'data', index:'data', width:200},
	  {name:'date', index:'date', width:200},
	],
	sortname: 'date',
	sortorder: 'asc',
	caption:'Search: ',
	viewrecords: true,
	loadonce: true,
	width: 750,
	forceFit: true,
	height:130,
	gridComplete: function() {
		searchColumn = grid.jqGrid('getCol','name',true) //needed for live filtering search
	}
  });

```



### Live Filtering


This is the simple code that allows us to live filter on the grid.  It will hide any id that doesn't match the string you're entering.  The search is case insensitive.

```javascript
//for live filtering search
jQuery('#gridsearch').keyup(function () {
	var searchString = jQuery(this).val().toLowerCase()
	jQuery.each(searchColumn,function() {
		if(this.value.toLowerCase().indexOf(searchString) == -1) {
			jQuery('#'+this.id).hide()
		} else {
			jQuery('#'+this.id).show()
		}
	})
})

```




### Known Issues


At the moment if the first row is hidden the grid becomes unaligned from its headers (and live resizing of the columns does not work).  This issue can be hidden by making all the headers the same width, but if a user resizes the problem will be visible again.  There is a [bug](http://github.com/tonytomov/jqGrid/issues#issue/26) open on github to correct the issue.
