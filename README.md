Draw Multiple Links (Paths) Between Two Nodes in D3.js Force-Directed-Layout
============================================================================

The original visualization of D3.js Force-Directed-Layout is suppose there is only one link between two nodes. 
If we want to draw a graph with multiple links between nodes, we need a way to change the radius of [svg path(arc)](http://www.w3.org/TR/SVG/paths.html#PathDataEllipticalArcCommands) elements representing the link.
Besides, it's better that the arc can increase its radius in a visually nice way. The subsequent path should gradually increase its radius to avoid overlapping and crossing.

**[Try Yourself!](http://jsfiddle.net/zhanghuancs/a2QpA/)**

Overview
--------
- Sort links by source id, then by target id, so that the links with same source id and target id can be in order.
- Add `linkindex` attribute to the node - indicating that the current link is the #linkindex of link among the total links
- Calculate the total number of links between two nodes and store the information for later use.
- Change the radius of arc path based on the total number of links between two nodes, and the current link's link index

Sort Links
-----------------------
The links with the same source id and target id might not be put together, sort the links by source id first, then by target id.

```javascript
function sortLinks()
{								
	data.links.sort(function(a,b) {
		if (a.source > b.source) 
		{
			return 1;
		}
		else if (a.source < b.source) 
		{
			return -1;
		}
		else 
		{
			if (a.target > b.target) 
			{
				return 1;
			}
			if (a.target < b.target) 
			{
				return -1;
			}
			else 
			{
				return 0;
			}
		}
	});
}	
```

Set Total Number of Links and Link Index
-----------------------------------------------------------------------------------------------------------
After sorting the links, the links with the same source id and target id are put next to each other. 
Then we need loop through the links and get the information of the total number of links between two nodes, and each link's link index.

- `mLinkNum` is an object used to store the total number of links between each of two nodes. 
   For example, is `mLinkNum = {"0,1":"3", "0,2":"5"}`, it means that there are 3 links between node 0 and node 1, there are 5 links between node 0 and node 2.
- `linkindex` is set to each link, indicating this link the #linkindex of link among the total links

These two variables will be used to calculate the radius for this arc path.

```javascript
//any links with duplicate source and target get an incremented 'linkindex'
function setLinkIndexAndNum()
{								
	for (var i = 0; i < data.links.length; i++) 
	{
		if (i != 0 &&
			data.links[i].source == data.links[i-1].source &&
			data.links[i].target == data.links[i-1].target) 
		{
			data.links[i].linkindex = data.links[i-1].linkindex + 1;
		}
		else 
		{
			data.links[i].linkindex = 1;
		}
		// save the total number of links between two nodes
		if(mLinkNum[data.links[i].target + "," + data.links[i].source] !== undefined)
		{
			mLinkNum[data.links[i].target + "," + data.links[i].source] = data.links[i].linkindex;
		}
		else
		{
			mLinkNum[data.links[i].source + "," + data.links[i].target] = data.links[i].linkindex;
		}
	}
}	
```

Calculate Arc Path Radius 
------------
The radius of each arc path between two nodes should be gradually increase. 
Use a simple formula `dr = dr/(1 + (1/lTotalLinkNum) * (d.linkindex - 1))` to set different but gradually increased radius to the link.

```javascript
path.attr("d", function(d) {
	var dx = d.target.x - d.source.x,
		dy = d.target.y - d.source.y,
		dr = Math.sqrt(dx * dx + dy * dy);
	// get the total link numbers between source and target node
	var lTotalLinkNum = mLinkNum[d.source.id + "," + d.target.id] || mLinkNum[d.target.id + "," + d.source.id];
	if(lTotalLinkNum > 1)
	{
		// if there are multiple links between these two nodes, we need generate different dr for each path
		dr = dr/(1 + (1/lTotalLinkNum) * (d.linkindex - 1));
	}	    
	// generate svg path
	return "M" + d.source.x + "," + d.source.y + 
		   "A" + dr + "," + dr + " 0 0 1," + d.target.x + "," + d.target.y + 
		   "A" + dr + "," + dr + " 0 0 0," + d.source.x + "," + d.source.y;	
});
```