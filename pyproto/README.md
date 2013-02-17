SNAPGraphConstruction
=====================

Idea: Provide a language and interface to convert raw data into meaningful graphs

Motivation: Most of the research in network analysis is based on pre-labeled graphs.
However, there may be many other ways of defining nodes and edges. Each definition could
potentially reveal important properties of the network.

Plan of Action:
1.  Define the language to do relational algebra queries on data.
2.  Write functions that help the user select parts of data that could be labeled nodes
    and edges and add to a graph

Assumptions:
1.  Data set includes schema of data
2.	Data comes in as .xml files 
3.	Data is of three basic types: float, string and dates

Issues:
1. 	TSV format not well defined, encode escape working with python
2.	Memory management
3.	Indexes? performance matters

Things to think about
	I. abstraction from sql level
		- functions like selectandjoin on two tables given attributes for selection and join
		Example: user calls MakeGraph(files with data, schema, optional type of graph expected)
			MakeGraph reads data from files into tables. Identifies common attributes to join on
			Decides logic to select nodes and edges. Calls 
			selectandjoin(table1, table2, selectattr1, selectattr2, joinattrs, condition1, condition2, groupbyattr)
			selectandjoin returns a table of possible nodes.
			select node pairs satisfying some condition as nodes and the pair as edges

		- functions to construct sql query based on what type of graph to make
		Example: User calls MakeGraph(files with data, schema, optional type of graph expected)
			MakeGraph reads data from files into tables. Constructs a sql query using select and join on different sets of
			attributes. Since this function is data-agnostic, it will try some obvious ones like primary keys. Similarly it will join on common attributes. It identifies common attributes using schema. It will also try multiple conditions for selection.
			It constructs multiple sql like queries and passes them to a query parser/optimizer. The optimizer will take a sql
			query and convert into a set of function calls select, join, group etc. that are logically equivalent to the query passed
			in. The optimizer will also suggest the best sequence of these function calls to enhance performance.

			Then MakeGraph calls the sequence of functions suggested by optimizer. It gets back a table like this:

			candidate node 1	candidate node 2	some simple aggregation metric
			1					1					5
			1					2					7
			1					2					5
			1					3					2
			2					3					6

			Say our logic is select the pairs of nodes with aggr metric >= 5.

			Then MakeGraph creates a new graph using 1, 2 and 3 as ndoes and [1,1], [1,2], [1,2] and [2,3] as edges.
			This is assuming self-loops and mutiedges are ok.

			I believe the result(s) of MakeGraphs could be fed to the analysis framework to test how good the graphs are and the feedback passed back to MakeGraph. That way MakeGraph would run multiple times to output the best graphs after some number
			of runs.
Next Steps:
1.	New graphs (content-based): use tags for posts; looking at graphs using some other datasets like Twitter, Citation networks
2.	Look at standard database primitives and implement
3.	Implementing date graph - implementing expand function
4.	Distance based graphs
5.	Draw a list of equivalent sequences of sql primitives that can be used to construct graphs


ISSUES:
1. 	select function should take multiple conditions. Likewise getTuples in table should support multiple conditions
	In order to have conditions take in the attribute on which the condition is
	applied, Condition is now constructed as (attr, op, value).
2. 	Projection should be done inside select so that we don't carry around too much data to join step. Renaming can be done 
	on the tables returned by select.
3. 	We need join conditions. Separate from Conditions. A JoinCondition should have the attr A in table 1 that match attr B
	in table 2.
4.	Content-based graphs are mostly based on distance metric, at the very least Jaccard distance.
5.	Additional primitives to implement: order by(sort), unique, bag intersection, bag union, bag difference
6	Equivalence laws:
	(I)	Selection:
		select(R, condition) union/difference select(S, condition) = select (R union/difference S, condition)
		select(R intersect S, condition) = select(R, condition) intersect S
		select(R join S, condition) = select(R, condition) join select(S, condition)
		unique(R join S) = unique(R) join unique(S)
		unique(select(R, condition)) = select(unique(R), condition)
		unique(group(R, attrs)) = group(R, attrs)
		select(R, cond1) join select(S, cond2) = select(R join S, cond1, cond2)
		

