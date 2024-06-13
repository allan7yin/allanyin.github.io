## Question 1

```Python
def FindNextStop(l, tank, D[1,2,...n]):
	# find the max next stop starting from l + D[l]
	localMax = l + D[l]
	localMaxIndex = l
	n = len(D)
	
	for i in range(l + 1, n):
	    if i + D[i] > localMax:
	        localMax = i + D[i]
	        localMaxIndex = i
	
	return localMaxIndex # this is where we want to drive to, no stops 
		
@Algo Min Gas Station Problem
def MinGasStops(D[1,2,...n]):
	sol = [D[1]]
	tank = D[1]
	l = 1

	while l < n:
		dest = FindNextStop(l, tank, D)
		l = dest
		tank = D[l]
		sol.append(D[l]) # since we are refueling here 
	
	return sol
```

  

## Question 2

2 ways I have thought of. First one is derived from min colours over intervals problem sorts by start time and uses a minHeap:

**⇒ Method 1**

```Python
@Algo Max k Colour Interval Problem 
def MaxIntervalColouring(k, intervals): # where intervals is list of pairs of form [start, finish]
	# let colours be from 1 ... k
	sort(intervals) # sort intervals by start time 
	h = new minHeap()
	sol = []
	
	for i in 1 ... k:
		h.insert([-1, i]) # [finish time, colour]

	for interval in intervals: # interval is of form [start, finish]
		start,finish = interval[0], interval[1]
		topColourFinsh, topColour = h.root() # earliest available colour 

		if topColourFinish <= start:
			sol.append([interval, topColour])
			# reset the time of the colour
			h.pop() # pops the used colour off the minHeap
			h.insert([finish, topColour])
		else:
			# no colour is available, this will be colourless 
			sol.append([interval, -1]) # let -1 indicate no colour 
	
	return sol	
```

  

**⇒ Method 2**

This one is based on the interval scheduling problem. So, we sort by end time, and maintain an AVL/ Balanced BST, where there are k nodes and each one represents an end time of one the intervals. Recall that when we only had one node, hence just choosing the largest subset of no overlaps, we kept track of the previous selected intervals end time, and we add an interval if its start time is after that end time, then, we change the end time. Given an interval, we look through the bst for the latest end time that is smaller than the start time of the current interval.

## Question 2

```Python
@Algo Max k Colour Interval Problem 
def MaxIntervalColouring(k, intervals): # where intervals is list of pairs of form [start, finish]
	# let colours be from 1 ... k
	sort(intervals) # sort intervals by end time in increasing order 
	sol = [] # solution array, contains (interval, colour #), we let -1 indicate no colour 
	
	bst = new BST() # create a balanced binary search tree, will be inserting up to k nodes inside -> (start, finish, colour)
	bst.insert(intervals[1])
	
	for i = 2 ... n
		treeMin = bst.min()
		if intervals[i].s < treeMin.f:
			if bst.size() < k:
				bst.insert(intervals[i])
				sol.append(intervals[i])
	
			else:
				n = bst.search(s) # find node in the tree with endtime < start of intervals[i] but as close to intervals[i] as possible 
				bst.delete(n)
				bst.insert(intervals[i])
				sol.append(intervals[i])

	return sol	
```

## Question 3

```Python
@Algo Min Sum of Photo Row Heights 
def MinPhotoRowHeights(photos, W): # where photos is list of photo of structure: [width, height], W is width constraint
	h = []
	w = []
	for photo in photos:
		w.append(photo[0])
		h.append(photo[1])
	dp = [0, h[1]] #  will hold minimum sum of row heights for the first 'i' photos, indexed from 1, 2 base cases added already

	# the general case for i >= 2
	for i = 2 .. n:
		# place onto a new row 
		dp[i] = dp[i-1] + h[i]
		currWidth, maxHeight, k = w[i], h[i], i-1
		while currWidth <= W and k > 0:
			maxHeight = max(maxHeight, h[k])
			dp[i] = min(dp[i], maxHeight + dp[k-1])
			currWidth += w[k]
			k -= 1
	
	return dp[n] # where this is the minimum sum of row heights for the n photos 
```

  

## Question 4 - o;ld

```Python
@Algo Max Client Priority Power Distribution 
def MaxClientPriorityPower(importance, demand, MA, MB):
	n = len(importance)
	dp[][][] # 3D array to store states of the dp 

	for i = 1 ... n:
		for ma = 0 ... MA:
			for mb = 0 ... MB:
			# if we both SA and SB have capacity to power i
				if demand[i] <= ma and demand[i] <= mb:
					dp[i][ma][mb] = max(demand[i] + dp[i-1][ma - demand[i]][mb],
															max(demand[i] + dp[i-1][ma][mb - demand[i]],
																	dp[i-1][ma][mb]))
	
				# if only SA has enough capacity to power i 
				elif demand[i] <= ma:
					dp[i][ma][mb] = max(demand[i] + dp[i-1][ma - demand[i]][mb],
															dp[i-1][ma][mb])
					
				# if only SB has enough capacity to power i 
				elif demand[i] <= mb:
					dp[i][ma][mb] = max(demand[i] + dp[i-1][ma][mb - demand[i]],
															dp[i-1][ma][mb])
	
				# neither has enough capacity
				else:
					dp[i][ma][mb] = dp[i-1][ma][mb]
			
	return dp[n][MA][MB] # where this is the maximum total importance over n clients 
```

## Question 4

```Python
@Algo Max Client Priority Power Distribution 
def MaxClientPriorityPower(importance, demand, MA, MB):
	n = len(importance)
	dp[][][] # 3D array to store states of the dp 
	dpStation[][][]

	for i = 1 ... n:
		for ma = 0 ... MA:
			for mb = 0 ... MB:
			# if we both SA and SB have capacity to power i
				if demand[i] <= ma and demand[i] <= mb:
				dp[i][ma][mb] = demand[i] + dp[i-1][ma - demand[i]][mb]
				dpStation[i][ma][mb] = "S_A"

				if demand[i] + dp[i-1][ma][mb - demand[i]] > dp[i][ma][mb]:
				    dp[i][ma][mb] = demand[i] + dp[i-1][ma][mb - demand[i]]
						dpStation[i][ma][mb] = "S_B"
				
				if dp[i-1][ma][mb] > dp[i][ma][mb]:
				    dp[i][ma][mb] = dp[i-1][ma][mb]
						dpStation[i][ma][mb] = "None"
					
				# if only SA has enough capacity to power i 
				elif demand[i] <= ma:
					if (demand[i] + dp[i-1][ma - demand[i]][mb] > dp[i-1][ma][mb]):
						dp[i][ma][mb] = demand[i] + dp[i-1][ma - demand[i]][mb]
						dpStation[i][ma][mb] = "S_A"
					else:
						dp[i][ma][mb] = dp[i-1][ma][mb]
						dpStation[i][ma][mb] = "None"
					
				# if only SB has enough capacity to power i 
				elif demand[i] <= mb:
					if (demand[i] + dp[i-1][ma][mb - demand[i]] > dp[i-1][ma][mb]):
						dp[i][ma][mb] = demand[i] + dp[i-1][ma][mb - demand[i]]
						dpStation[i][ma][mb] = "S_B"
					else:
						dp[i][ma][mb] = dp[i-1][ma][mb]
						dpStation[i][ma][mb] = "None"
	
				# neither has enough capacity
				else:
					dp[i][ma][mb] = dp[i-1][ma][mb]
					dpStation[i][ma][mb] = "None"

	return dp[n][MA][MB] # where this is the maximum total importance over n clients 
```

Below, we modify this to accompany the second part of this question. That is, show optimal assignment of clients to $S_1$﻿ and $S_2$﻿

```Python
@Algo Get Assignmens For Max Client Priority Power Distribution 
def GetAssignments(dpStation, demand, n, MA, MB):
	# to obtain the respective lists, we work backwards from dp[n][MA][MB] and construct our lists 
	solA = []
	solB = []

	ma = MA
	mb = MB
	
	while n > 0:
		if dpStation[n][ma][mb] == "S_A":
			n -= 1
			ma -= demand[n]
			solA.append(n)
		elif dpStation[n][ma][mb] == "S_B":
			n -=1 
			mb -= demand[n]
			solB.append(n)
		else:
			n -= 1

	return solA, solB
			
```