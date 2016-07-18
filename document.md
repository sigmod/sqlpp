# The SQL++ Query Language

## <a id="toc">Table of Contents</a> ##

* [1. Introduction](#Introduction)
* [2. Expressions](#Expressions)  
  * [Primary expressions](#Primary_expressions)
    * [Literals](#Literals)
    * [Variable references](#Variable_references)
    * [Parenthesized expressions](#Parenthesized_expressions)
    * [Function call expressions](#Function_call_expressions)
    * [Constructors](#Constructors)
  * [Path expressions](#Path_expression)
    * [Field access expressions](#Field_access_expressions)
    * [Index access expressions](#Index_access_expressions)
  * [Operator expressions](#Operator_expressions)
    * [Arithmetic operators](#Arithmetic_operators)
    * [Collection operators](#Collection_operators)
    * [Comparison operators](#Comparison_operators)
    * [Logical operators](#Logical_Operators)
  * [Conditional expressions](#Conditional_expressions)
  * [Quantified expressions](#Quantified_expressions)
* [3. Queries](#Queries)
  * [Select statement](#Select_statement)
  * [Select clause](#Select_clause)
    * [Select element/value/raw](#Select_element)
    * [SQL-style select](#SQL_select)
    * [Select *](#Select_star)
    * [Select distinct](#Select_distinct)
    * []
  * [Where clause](#Where_clause)
  * [Unnest clause](#Unnest_clause)
    * [Inner unnests](#Innner_unnests)
    * [Left outer unnests](#Left_outer_unnests)
    * [Expressing joins using unnests](#Expressing_joins_using_unnests)
  * [From clause](#From_clause)
    * [Binding expressions](#Binding_expressions)
    * [Multiple from terms](#Multiple_from_terms)
    * [Expressing joins using from terms](#Express_joins_using_from_terms)
  * [Join clause](#Join Clause)
    * [Inner Joins](#Inner_joins)
    * [Left Outer Joins](#Left_outer_joins)
  * [Group By clause](#Group_By)
  * [Order By clause](#ORDER_BY)
  * [Limit clause](#Limit)
  * [SQL++ Vs. SQL-92](#Vs_SQL-92)
* [4. DDL and DML Statements](#DDL_and_DML_Statements)

# <a id="Introduction">1. Introduction</a><font size="4">

This document is intended as a reference guide to the full syntax and semantics of the SQL++ Query Language, a language for talking to AsterixDB.

New AsterixDB users are encouraged to read and work through the (friendlier) guide "AsterixDB 101: An ADM and SQL++ Primer" before attempting to make use of this document. In addition, readers are advised to read and understand the Asterix Data Model (ADM) reference guide since a basic understanding of ADM concepts is a prerequisite to understanding SQL++. In what follows, we detail the features of the SQL++ language in a grammar-guided manner: we list and briefly explain each of the productions in the SQL++ grammar, offering examples for clarity in cases where doing so seems needed or helpful.

# <a id="Expressions">Expressions

    Expression ::= OperatorExpression | ConditionExpression | QuantifiedExpression

Each SQL++ expression returns zero or more Asterix Data Model (ADM) instances. There are three major kinds of expressions in SQL++. At the topmost level, an SQL++ expression can be an OperatorExpression (similar to a mathematical expression), an ConditionalExpression (to choose between alternative values), or a QuantifiedExpression (which yields a boolean value). Each will be detailed as we explore the full SQL++ grammar.

## <a id="Primary_expressions">Primary Expressions

    PrimaryExpr ::= Literal
                  | VariableReference
                  | ParenthesizedExpression
                  | FunctionCallExpression
                  | Constructor

The most basic building block for any SQL++ expression is PrimaryExpression. This can be a simple literal (constant) value, a reference to a query variable that is in scope, a parenthesized expression, a function call,  a newly constructed list of ADM instances, or a newly constructed ADM record.

### <a id="Literals">Literals

    Literal        ::= StringLiteral
                       | IntegerLiteral
                       | FloatLiteral
                       | DoubleLiteral
                       | <NULL>
                       | <MISSING>
                       | <TRUE>
                       | <FALSE>
    QuotedString   ::= "\`" (<ESCAPE_APOS> | ~["\'"])* "\`"
    StringLiteral  ::= "\'" (<ESCAPE_APOS> | ~["\'"])* "\'"
                       | "\"" (<ESCAPE_APOS> | ~["\'"])* "\""
    <ESCAPE_QUOT>  ::= "\\\""
    <ESCAPE_APOS>  ::= "\\\'"
    IntegerLiteral ::= <DIGITS>
    <DIGITS>       ::= ["0" - "9"]+
    FloatLiteral   ::= <DIGITS> ( "f" | "F" )
                     | <DIGITS> ( "." <DIGITS> ( "f" | "F" ) )?
                     | "." <DIGITS> ( "f" | "F" )
    DoubleLiteral  ::= <DIGITS>
                     | <DIGITS> ( "." <DIGITS> )?
                     | "." <DIGITS>

Literals (constants) in SQL++ can be strings, integers, floating point values, double values, boolean constants, special constant values like `NULL` and `MISSING`. The `NULL` value is identifical to nulls in the relational query language SQL, which mean the value of a field is unknown. The `MISSING` value is only meaningful in the context of field accesses and it means a field does not exist in a record at all.

The following are some simple examples of SQL++ literals.

#### Examples

    'a string'
    "test string"
    42

Different from standard SQL, double quotes are the same as single quotes and could also be used for string literals in SQL++. Backticks, i.e., \`id\`, are for delimited identifiers.

### <a id="Variable_references">Variable References

    VariableReference ::= <IDENTIFIER>|<QuotedString>
    <IDENTIFIER>  ::= <LETTER> (<LETTER> | <DIGIT> | "_" | "$")*
    <LETTER>    ::= ["A" - "Z", "a" - "z"]

A variable in SQL++ can be bound to any legal ADM value. A variable reference refers to the value to which an in-scope variable is bound. (E.g., a variable binding may originate from one of the `FROM`, `WITH` or `LET` clauses of a select statement or from an input parameter in the context of a function body.)

#### Examples

    tweet
    id

### <a id="Parenthesized_expressions">Parenthesized expressions

    ParenthesizedExpression ::= "(" Expression ")" | Subquery

An expression can be parenthesized, which usually is used for controlling the precedence order. In SQL++, a subquery is also an parenthesized expression.

The following expression is evaluated to value 2.

##### Example

    ( 1 + 1 )

### <a id="Function_call_expressions">Function call expressions

    FunctionCallExpression ::= FunctionName "(" ( Expression ( "," Expression )* )? ")"

Functions are included in SQL++, like most languages, as a way to package useful functionality or to componentize complicated or reusable SQL++ computations. A function call is a legal SQL++ query expression that represents the ADM value resulting from the evaluation of its body expression with the given parameter bindings; the parameter value bindings can themselves be any SQL++ expressions.

The following example is a (built-in) function call expression whose value is 8.

#### Example

    length('a string')

### <a id="Constructors">Constructors

    ListConstructor          ::= OrderedListConstructor | UnorderedListConstructor
    OrderedListConstructor   ::= "[" ( Expression ( "," Expression )* )? "]"
    UnorderedListConstructor ::= "{{" ( Expression ( "," Expression )* )? "}}"
    RecordConstructor        ::= "{" ( FieldBinding ( "," FieldBinding )* )? "}"
    FieldBinding             ::= Expression ":" Expression

A major feature of SQL++ is its ability to construct new ADM data instances. This is accomplished using its constructors for each of the major ADM complex object structures, namely lists (ordered or unordered) and records. Ordered lists are like JSON arrays, while unordered lists have bag (multiset) semantics. Records are built from attributes that are field-name/field-value pairs, again like JSON. (See the AsterixDB Data Model document for more details on each.)

The following examples illustrate how to construct a new ordered list with 3 items, a new unordered list with 4 items, and a new record with 2 fields, respectively. List elements can be homogeneous (as in the first example), which is the common case, or they may be heterogeneous (as in the second example). The data values and field name values used to construct lists and records in constructors are all simply SQL++ expressions. Thus the list elements, field names, and field values used in constructors can be simple literals (as in these three examples) or they can come from query variable references or even arbitrarily complex SQL++ expressions.

#### Examples

    [ 'a', 'b', 'c' ]

    {{ 42, 'forty-two', 'AsterixDB!', 3.14f }}

    {
      'project name': 'AsterixDB'
      'project members': {{ 'vinayakb', 'dtabass', 'chenli' }}
    }

### <a id="Path_expressions">Path expressions

    ValueExpr ::= PrimaryExpr ( Field | Index )*
    Field     ::= "." Identifier
    Index     ::= "[" ( Expression | "?" ) "]"

Components of complex types in ADM are accessed via path expressions. Path access can be applied to the result of an SQL++ expression that yields an instance of  a complex type, e.g., a record or list instance. For records, path access is based on field names. For ordered lists, path access is based on (zero-based) array-style indexes. SQL++ also supports an "I'm feeling lucky" style index accessor, [?], for selecting an arbitrary element from an ordered list. Attempts to access non-existent fields or out-of-bound list elements produce a missing.

The following examples illustrate field access for a record, index-based element access for an ordered list, and also a composition thereof.

#### Examples

    ({'list': [ 'a', 'b', 'c']}).list

    (['a', 'b', 'c'])[2]

    ({ 'list': [ 'a', 'b', 'c']}).list[2]

### <a id="Operator_expressions">Operator expressions
    
Operators perform a specific operation on the input values or expressions. AsterixDB SQL++ provides a full set of operators that you can use within its statements. Here are the categories of operators:

* [Arithmetic operators](#Arithmetic_operators), to perform basic mathematical operations;
* [Collection operators](#Collection_operators), to evaluate expressions on collections or objects;
* [Comparison operators](#Comparison_operators), to compare two expressions;
* [Logical Operators](#Logical_operators), to combine operators using Boolean logic.

The following table summarizes the precedence order (from higher to lower) of all operators:

| Operator                                                                    | Operation |
|-----------------------------------------------------------------------------|-----------|
| +, -, EXISTS, NOT EXISTS                                                    |  identity, negation  |
| ^                                                                           |  exponentiation  |
| *, /                                                                        |  addition, subtraction |
| IS NULL, IS NOT NULL, IS MISSING, IS NOT MISSING, <br>IS UNKNOWN, IS NOT UNKNOWN| comparison |
| =, !=, <, >, <=, >=, LIKE, NOT LIKE, IN, NOT IN                             | comparison  |
| NOT                                                                         | logical negation |
| AND                                                                         | conjunction |
| OR                                                                          | disjunction |

### <a id="Arithmetic operators">Arithmetic operators
Arithemtic operators are used to exponentiate, negate, add, subtract, multiply, and divide numeric values.
 
| Operator |  Purpose                                                                       | Example    |
|----------|--------------------------------------------------------------------------------|------------|
| +, -     |  When they are unary operators, these denote a <br>positive or negative expression.| SELECT -1; |
|          |  When they are binary operators, these add or substract.                       | SELECT 1+2; SELECT 2-1;   |           
| *, /     |  Multiply, divide.                                                             | SELECT 3*2; SELECT 4/2.0; |
| ^        |  Exponentiation.                                                               | SELECT 3^5;       |

### <a id="Collection operators">Collection operators
Collection operators are used for membership tests (IN, NOT IN) or empty collection tests (EXISTS, NOT EXISTS).

| Operator   |  Purpose                                     | Example    |
|------------|----------------------------------------------|------------|
| IN         |  Membership test.                            | SELECT * FROM TweetMessages tm <br>WHERE tm.user.lang IN ["en", "de"]; |
| NOT IN     |  Non-membership test.                        | SELECT * FROM TweetMessages tm <br>WHERE tm.user.lang NOT IN ["en"]; |
| EXISTS     |  Check whether a collection is not empty.    | SELECT * FROM TweetMessages tm <br>WHERE EXISTS tm.referedTopics; |
| NOT EXISTS |  Check whether a collection is empty.        | SELECT * FROM TweetMessages tm <br>WHERE NOT EXISTS tm.referedTopics; |


### <a id="Comparison operators">Comparison operators
Comparison operators are used to compare values.

| Operator       |  Purpose                                   | Example    |
|----------------|--------------------------------------------|------------|
| IS NULL        |  Test if a value is null.                  | SELECT * FROM TweetMessages tm <br>WHERE tm.user.name IS NULL; |
| IS NOT NULL    |  Test if a value is not null.              | SELECT * FROM TweetMessages tm <br>WHERE tm.user.name IS NOT NULL; | 
| IS MISSING     |  Test if a value is missing.               | SELECT * FROM TweetMessages tm <br>WHERE tm.user.name IS MISSING; |
| IS NOT MISSING |  Test if a value is not missing.           | SELECT * FROM TweetMessages tm <br>WHERE tm.user.name IS NOT MISSing;|
| IS UNKNOWN     |  Test if a value is null or missing.       | SELECT * FROM TweetMessages tm <br>WHERE tm.user.name IS UNKNOWN; | 
| IS NOT UNKNOWN |  Test if a value is not null nor missing.  | SELECT * FROM TweetMessages tm <br>WHERE tm.user.name IS NOT UNKNOWN;| 
| =              |  Equality test.                            | SELECT * FROM TweetMessages tm <br>WHERE tm.tweetid=10; |
| !=             |  Inequality test.                          | SELECT * FROM TweetMessages tm <br>WHERE tm.tweetid!=10;|
| <              |  Less than.                                | SELECT * FROM TweetMessages tm <br>WHERE tm.tweetid<10; |
| >              |  Greater than.                             | SELECT * FROM TweetMessages tm <br>WHERE tm.tweetid>10; |
| <=             |  Less than or equal to.                    | SELECT * FROM TweetMessages tm <br>WHERE tm.tweetid<=10; |
| >=             |  Greater than or equal to.                 | SELECT * FROM TweetMessages tm <br>WHERE tm.tweetid>=10; |
| LIKE           |  Test if the left side matches a<br> pattern defined at the right<br> side. In the pattern,  "%" matches  <br>any string while "_" matches <br> any character. | SELECT * FROM TweetMessages tm <br>WHERE tm.user.name LIKE "%Giesen%";|
| NOT LIKE       |  Test if the left side does not <br>match a pattern defined at the right<br> side. In the pattern,  "%" matches <br>any string while "_" matches <br> any character. | SELECT * FROM TweetMessages tm <br>WHERE tm.user.name NOT LIKE "%Giesen%";| 

### <a id="Logical_operators">Logical operators
Logical operators perform logical `inverse`, `and`, `or` operatorions for boolean values, `null` or `missing`.

| Operator |  Purpose                                   | Example    |
|----------|-----------------------------------------------------------------------------|------------|
| NOT      |  Returns true if the following condition is false, otherwise returns false. | SELECT NOT TRUE;  |
| AND      |  Returns true if both branches are true, otherwise returns false.           | SELECT TRUE AND FALSE; | 
| OR       |  Returns true if one branch is true, otherwise returns false.               | SELECT FALSE OR FALSE; |

The following table is the truth table for `AND` and `OR`.

| A  | B  | A AND B  | A OR B |
|----|----|----------|--------|
| TRUE | TRUE | TRUE | TRUE |
| TRUE | FLASE | FALSE | TRUE |
| TRUE | NULL | NULL | TRUE |
| TRUE | MISSING | MISSING | TRUE |
| FALSE | FALSE | FALSE | FALSE |
| FALSE | NULL | FALSE | NULL |
| FLASE | MISSING | FALSE | MISSING |
| NULL | NULL | NULL | NULL |
| NULL | MISSING | MISSING | NULL |
| MISSING | MISSING | MISSING | MISSING |

The following table demonstrates the results of `NOT` on possible inputs.

| A  | NOT A |
|----|----|
| TRUE | FALSE |
| FALSE | TRUE |
| NULL | NULL |
| MISSING | MISSING | 

### <a id="Conditional_expressions">Conditional expressions

    IfThenElse ::= <IF> "(" Expression ")" <THEN> Expression <ELSE> Expression

A conditional expression is useful for choosing between two alternative values based on a
boolean condition.  If its first (<IF>) expression is true, its second (<THEN>) expression's
value is returned, and otherwise its third (<ELSE>) expression is returned.

The following example illustrates the form of a conditional expression.
#### Example

    IF (2 < 3) THEN "yes" ELSE "no"

### <a id="Quantified_expressions">Quantified expressions

    QuantifiedExpression ::= ( <SOME> | <EVERY> ) Variable <IN> Expression ( "," Variable "in" Expression )*
                             <SATISFIES> Expression

Quantified expressions are used for expressing existential or universal predicates involving the elements of a collection.

The following pair of examples illustrate the use of a quantified expression to test that every (or some) element in the set [1, 2, 3] of integers is less than three. The first example yields `FALSE` and second example yields `TRUE`.

It is useful to note that if the set were instead the empty set, the first expression would yield `TRUE` ("every" value in an empty set satisfies the condition) while the second expression would yield `FALSE` (since there isn't "some" value, as there are no values in the set, that satisfies the condition).

##### Examples

    EVERY x IN [ 1, 2, 3 ] SATISFIES x < 3
    SOME x IN [ 1, 2, 3 ] SATISFIES x < 3


# <a id="Queries">2. Queries</a>

A SQL++ query can be any legal SQL++ expression or Select statment. A query should always end with a semicolon.

    Query ::= (Expression | SelectStatement) ";"

##  <a id="Select_statements">Select statements

The following BNFs (Backus–Naur Forms) show the grammar of select statements in AsterixDB SQL++.

    SelectStatement    ::=	( WithClause )? SelectSetOperation (OrderbyClause )? ( LimitClause )?
    SelectSetOperation ::=	 SelectBlock ( (<UNION> | <INTERSECT> | <EXCEPT>)  ( <ALL>)? ( SelectBlock | Subquery ) )*
    Subquery	       ::=	"(" SelectStatement ")"
    
    SelectBlock	       ::=	SelectClause ( FromClause ( WithClause )?)? (WhereClause )? 
                            ( GroupbyClause ( LetClause )? ( HavingClause )? )? 
                            |
                            FromClause ( WithClause )? ( WhereClause )? ( GroupbyClause ( WithClause )? 
                            ( HavingClause )? )? SelectClause 
    
    SelectClause       ::=	<SELECT> ( <ALL> | <DISTINCT> )? ( SelectRegular | SelectElement )
    SelectRegular      ::=	Projection ( "," Projection )*
    SelectElement      ::=	( <RAW> | <ELEMENT> | <VALUE> ) Expression
    Projection	       ::=	( Expression ( <AS> )? Identifier | "*" )
  
    FromClause	       ::=	<FROM> FromTerm ( "," FromTerm )*
    FromTerm	       ::=	Expression (( <AS> )? Variable)? ( <AT> Variable )? 
                            ( ( JoinType )? ( JoinClause | UnnestClause ) )*
    
    JoinClause	       ::=	<JOIN> Expression (( <AS> )? Variable)? (<AT> Variable)? <ON> Expression
    UnnestClause       ::=	( <UNNEST> | <CORRELATE> | <FLATTEN> ) Expression ( <AS> )? Variable ( <AT> Variable )?
    JoinType	       ::=	( <INNER> | <LEFT> ( <OUTER> )? )
	
    WithClause	       ::=	<WITH> WithElement ( "," WithElement )*
    LetClause	       ::=	(<LET> | <LETTING>) LetElement ( "," LetElement )*
    LetElement	       ::=	Variable "=" Expression
    WithElement	       ::=	Variable <AS> Expression
  
    WhereClause	       ::=	<WHERE> Expression
    
    GroupbyClause      ::=	<GROUP> <BY> ( Expression ( (<AS>)? Variable )? ( "," Expression ( (<AS>)? Variable )? )* 
                            (<GROUP> <AS> Variable
                              ("(" Variable <AS> VariableReference ("," Variable <AS> VariableReference )* ")")?
                            )?
    HavingClause       ::=	<HAVING> Expression
    
    OrderbyClause      ::=	<ORDER> <BY> Expression ( <ASC> | <DESC> )? ( "," Expression ( <ASC> | <DESC> )? )*
    LimitClause	       ::=	<LIMIT> Expression ( <OFFSET> Expression )?

In this section, we will use two collections, `FacebookUsers` and `FacebookMessages`, as running examples to explain select queries. The contents of the two example datasets are shown as follows.

`FacebookUsers` collection:

    {"id":1,"alias":"Margarita","name":"MargaritaStoddard","user-since":datetime("2012-08-20T10:10:00"),"friend-ids":{{2,3,6,10}},"employment":[{"organization-name":"Codetechno","start-date":date("2006-08-06")},{"organization-name":"geomedia","start-date":date("2010-06-17"),"end-date":date("2010-01-26")}]}
    {"id":2,"alias":"Isbel","name":"IsbelDull","user-since":datetime("2011-01-22T10:10:00"),"friend-ids":{{1,4}},"employment":[{"organization-name":"Hexviafind","start-date":date("2010-04-27")}]}
    {"id":3,"alias":"Emory","name":"EmoryUnk","user-since":datetime("2012-07-10T10:10:00"),"friend-ids":{{1,5,8,9}},"employment":[{"organization-name":"geomedia","start-date":date("2010-06-17"),"end-date":date("2010-01-26")}]}

`FacebookMessages` collection:

    {"message-id":2,"author-id":1,"in-response-to":4,"sender-location":point("41.66,80.87"),"message":" dislike iphone its touch-screen is horrible"}
    {"message-id":3,"author-id":2,"in-response-to":4,"sender-location":point("48.09,81.01"),"message":" like samsung the plan is amazing"}
    {"message-id":4,"author-id":1,"in-response-to":2,"sender-location":point("37.73,97.04"),"message":" can't stand at&t the network is horrible:("}
    {"message-id":8,"author-id":1,"in-response-to":11,"sender-location":point("40.33,80.87"),"message":" like verizon the 3G is awesome:)"}
    {"message-id":6,"author-id":2,"in-response-to":1,"sender-location":point("31.5,75.56"),"message":" like t-mobile its platform is mind-blowing"}
    {"message-id":10,"author-id":1,"in-response-to":12,"sender-location":point("42.5,70.01"),"message":" can't stand motorola the touch-screen is terrible"}
    {"message-id":11,"author-id":1,"in-response-to":1,"sender-location":point("38.97,77.49"),"message":" can't stand at&t its plan is terrible"}

## <a id="Select_clause">Select clause
A select clause always return a collection, no matter the enclosing select statement is a top-level query or a subquery.

### <a id="Select_element">Select element/value/raw
`SELECT ELEMENT expression` returns a collection that consists of evaluation results of the expression, one per binding tuple. 
The following example shows a query that selects and returns one user from the table FacebookUsers.

#### Example

    SELECT ELEMENT user
    FROM FacebookUsers user
    WHERE user.id = 1;

It returns:

    [
      { "id": 1, "alias": "Margarita", "name": "MargaritaStoddard", "user-since": datetime("2012-08-20T10:10:00.000Z"), "friend-ids": {{ 2, 3, 6, 10 }}, "employment": [ { "organization-name": "Codetechno", "start-date": date("2006-08-06") }, { "organization-name": "geomedia", "start-date": date("2010-06-17"), "end-date": date("2010-01-26") } ] }
    ]

### <a id="SQL_select">SQL-style select
In SQL++, all traditional SQL-style select clauses could be expressed by `SELECT ELEMENT`.  For example, `SELECT exprA AS fieldA, exprB AS fieldB` is a syntactic suger of `SELECT ELEMENT { 'fieldA': expr1, 'fieldB': exprB }`. 

    SELECT user.alias user_alias, user.name user_name
    FROM FacebookUsers user
    WHERE user.id = 1;
    
It returns:

    [
      {"user_alias":"Margarita","user_name":"MargaritaStoddard"}
    ]

### <a id="SQL_star">Select *      
In SQL++,  `Select *` returns a nested record for each input binding tuple, where each field in the record has its field name being the name of a binding variable generated by either the `FROM` clause or `GROUP BY` clause in the current enclosing select statement, and has its field value being the value of the binding variable.

#### Example
   
    SELECT *
    FROM FacebookUsers user;

Since `user` is the only binding varibale generated in the `FROM` clause, it returns:

    [
      { "user": { "id": 1, "alias": "Margarita", "name": "MargaritaStoddard", "user-since": datetime("2012-08-20T10:10:00.000Z"), "friend-ids": {{ 2, 3, 6, 10 }}, "employment": [ { "organization-name": "Codetechno", "start-date": date("2006-08-06") }, { "organization-name": "geomedia", "start-date": date("2010-06-17"), "end-date": date("2010-01-26") } ] } },
      { "user": { "id": 2, "alias": "Isbel", "name": "IsbelDull", "user-since": datetime("2011-01-22T10:10:00.000Z"), "friend-ids": {{ 1, 4 }}, "employment": [ { "organization-name": "Hexviafind", "start-date": date("2010-04-27") } ] } }
    ]

### <a id="Select_distinct">Select distinct
`DISTINCT` is used to eliminate duplicate items in a result set. The following example shows how SQL++ `DISTINCT` keyword works.

#### Example

    SELECT DISTINCT * FROM [1, 2, 2, 3] AS foo;

It returns:

    [ 
      { "foo": 1 },
      { "foo": 2 },
      { "foo": 3 }
    ]

## <a id="Unnest_clause">Unnest clause
For each input tuple, Unnest clause flatterns a expression that returns to a collection value into each element value and produces multiple copies of the input tuple, each of which contains a flattern element value of the collection.

### <a id="Inner_unnest">Inner unnest
The next example shows a query that retrieves the organizations that the selected user has worked in, using the `UNNEST` clause to unnest the nested collection `employment` in the user's record.

#### Example

    SELECT user.id user_id, employment.`organization-name` org_name
    FROM FacebookUsers AS user
    UNNEST user.employment AS employment
    WHERE user.id = 1;

It returns:

    [
      { "user_id": 1, "org_name": "Codetechno" },
      { "user_id": 1, "org_name": "geomedia" }
    ]

Note that `UNNEST` has the "inner" semantics --- if a user does not have any employment history, the tuple corresponding to the user will not be emitted in the result.

### <a id="Left_outer_unnest">Left outer unnest
`LEFT OUTER UNNEST` has the "left outer" semantics. For example, field `foo` does not exist in the record for the user with id being 1, but the returned result set still contains the user's id.

    SELECT user.id user_id, employment.`organization-name` org_name
    FROM FacebookUsers AS user
    LEFT OUTER UNNEST user.foo AS employment
    WHERE user.id = 1;

It returns:

    [ 
      { "user_id": 1 }
    ]

### <a id="Express_joins_using_unnests">Expressing joins using unnests
The next example shows a query that joins two tables, FacebookUsers and FacebookMessages, returning user/message pairs. The results contain one record per pair, with result records containing the user's name and an entire message. The query can be seen as "for each Facebook user, unnest the entire `FacebookMessages` collection and then filters the output with condition `message.`\``author-id`\``= user.id`".  Of course, the underlying AsterixDB query processor will generate a query plan using hash join to evaluate the query since the condition is based on equality of fields from `FacebookUsers` and `FacebookMessages`.

#### Example

    SELECT user.name uname, message.message message
    FROM FacebookUsers user
    UNNEST FacebookMessages message
    WHERE message.`author-id` = user.id;

It returns:

    [
      { "uname": "MargaritaStoddard", "message": " can't stand at&t its plan is terrible" },
      { "uname": "MargaritaStoddard", "message": " dislike iphone its touch-screen is horrible" },
      { "uname": "MargaritaStoddard", "message": " can't stand at&t the network is horrible:(" },
      { "uname": "MargaritaStoddard", "message": " like verizon the 3G is awesome:)" },
      { "uname": "MargaritaStoddard", "message": " can't stand motorola the touch-screen is terrible" },
      { "uname": "IsbelDull", "message": " like t-mobile its platform is mind-blowing" },
      { "uname": "IsbelDull", "message": " like samsung the plan is amazing" }
    ]  

Similarly, the above query can also expressed as:

    SELECT user.name uname, message.message message
    FROM FacebookUsers user 
    UNNEST (	 
    	SELECT ELEMENT message
    	FROM FacebookMessages message
    	WHERE message.`author-id` = user.id
    ) AS message;

The query could read as "for each Facebook user, unnest the filtered `FacebookMessages` sub-collection with condition `message.`\``author-id`\``= user.id`". 

## <a id="From_clause">From clause
From clause is used for iterating over collections.

### <a id="Binding_expressions">Binding expressions
In SQL++, in addition to stored collections (i.e., tables), a `FROM` clause can iterate over any intermeiate collection returned by any valid SQL++ expression.

#### Example

    SELECT ELEMENT foo
    FROM [1, 2, 2, 3] AS foo
    WHERE foo>2;

It returns:

    [
      3
    ]

### <a id="Multiple_from_terms">Multiple from terms
SQL++ allows correlations among different from terms, i.e., a right side `FROM` binding expression can refer to variables defined on its left side. A equivalent query to the unnesting query above could be rewritten as:

    SELECT user.id user_id, employment.`organization-name` org_name 
    FROM FacebookUsers AS user, user.employment AS employment
    WHERE user.id = 1;

In general, query string like `expr1 AS v1 UNNEST expr2 AS v2` is equivalent to `expr1 AS v1, expr2 AS v2`.

### <a id="Expressing_joins_using_from_terms">Expressing joins using from terms
Replacing `UNNEST` with `,` the same join intent of the `UNNEST`-based join queries above could be expressed by (the first one is one of the SQL-92 ways to express the join intent):

    SELECT user.name uname, message.message message
    FROM FacebookUsers user, FacebookMessages message
    WHERE message.`author-id` = user.id;

or

    SELECT user.name uname, message.message message
    FROM FacebookUsers user, 
      (	 
    	SELECT ELEMENT message
    	FROM FacebookMessages message
    	WHERE message.`author-id` = user.id
      ) AS message;

## <a id="Join_clause">Join clause
AsterixDB SQL++ supports both inner joins and left outer joins.

### <a id="Inner_joins">Inner joins
Using join clause, the same join intent described above can be expressed as:

    SELECT user.name uname, message.message message
    FROM FacebookUsers user
    JOIN FacebookMessages message
    ON message.`author-id` = user.id;

Note that all the five appearances described so far for the same join intent yeld equivalently efficeint query execution plans generated by AsterixDB query optimizer. 

### <a id="Left_outer_joins">Left outer joins
SQL++ supports left outer join. The following query is an example:

    SELECT user.name uname, message.message message
    FROM FacebookUsers user
    LEFT OUTER JOIN FacebookMessages message
    ON message.`author-id` = user.id;

It returns

    [ 
      { "uname": "MargaritaStoddard", "message": " like verizon the 3G is awesome:)" },
      { "uname": "MargaritaStoddard", "message": " can't stand motorola the touch-screen is terrible" },
      { "uname": "MargaritaStoddard", "message": " can't stand at&t its plan is terrible" },
      { "uname": "MargaritaStoddard", "message": " dislike iphone its touch-screen is horrible" },
      { "uname": "MargaritaStoddard", "message": " can't stand at&t the network is horrible:(" },
      { "uname": "IsbelDull", "message": " like samsung the plan is amazing" },
      { "uname": "IsbelDull", "message": " like t-mobile its platform is mind-blowing" },
      { "uname": "EmoryUnk" }
    ]
   
Note that for non-matched left-side input tuples, SQL++ produces a `MISSING` for the right-side binding variables. That explains why the last record in the result does not have a `message` field. This is different from standard which fills `NULL`s for the right-side of non-matches. The reason is that by nature, for non-matches in the join results, the fields from the right-side are "missing".

The left-outer join query can also be expressed using `LEFT OUTER UNNEST`:

    SELECT user.name uname, message.message message
    FROM FacebookUsers user
    LEFT OUTER UNNEST (	 
    	SELECT ELEMENT message
    	FROM FacebookMessages message
    	WHERE message.`author-id` = user.id
      ) AS message;

In general, all join queries could be expressed by `UNNEST` clauses and all left outer join queries could be expressed by `LEFT OUTER UNNESTs`.

## <a id="Group_by_clause">Group By clause
The next example illustrates the use of the `GROUP BY` clause in SQL++. After the `GROUP BY` clause in the query,  the existing variables before the clause will each contain a collection of items following the `GROUP BY` clause; the collected items are the values that the source variable was bound to in the tuples that formed the group.

#### Example

      SELECT FacebookMessages.`author-id`,  COUNT(FacebookMessages.message) msgCount
      FROM FacebookMessages
      GROUP BY FacebookMessages.`author-id`;

It returns:

      [ 
        { "msgCount": 5, "author-id": 1 },
        { "msgCount": 2, "author-id": 2 }
      ]

## <a id="Order_by_clause">Order By clause
The following example returns all `FacebookUsers` ordered by their friend numbers. When ordering, `MISSING` and `NULL` is treated as being smaller than any other value if `MISSING` or `NULL`s are encountered in the ordering key(s), and `MISSING` is treated as smaller than `NULL`.

#### Example

      SELECT *
      FROM FacebookUsers AS user
      ORDER BY len(user.`friend-ids`) ASC;

It returns:

    [
      { "user": { "id": 2, "alias": "Isbel", "name": "IsbelDull", "user-since": datetime("2011-01-22T10:10:00.000Z"), "friend-ids": {{ 1, 4 }}, "employment": [ { "organization-name": "Hexviafind", "start-date": date("2010-04-27") } ] } },
      { "user": { "id": 1, "alias": "Margarita", "name": "MargaritaStoddard", "user-since": datetime("2012-08-20T10:10:00.000Z"), "friend-ids": {{ 2, 3, 6, 10 }}, "employment": [ { "organization-name": "Codetechno", "start-date": date("2006-08-06") }, { "organization-name": "geomedia", "start-date": date("2010-06-17"), "end-date": date("2010-01-26") } ] } }
    ] 

## <a id="Limit_clause">Limit clause
Limit clause is used to bound the result set to a constant size.
The use of the `LIMIT` clause is illustrated in the next example.

#### Example

      SELECT *
      FROM FacebookUsers AS user
      ORDER BY len(user.`friend-ids`) ASC
      LIMIT 1;

It returns:

    [
      { "user": { "id": 2, "alias": "Isbel", "name": "IsbelDull", "user-since": datetime("2011-01-22T10:10:00.000Z"), "friend-ids": {{ 1, 4 }}, "employment": [ { "organization-name": "Hexviafind", "start-date": date("2010-04-27") } ] } }
    ]


## <a id="Subqueries">Subqueries
In SQL++, an arbitrary subquery can appear at any place where an expression could appear. Different from SQL,  subqueries in `Projection`s or any boolean predicates are not restrained to return singleton, single-column relations, instead, they can return arbitrary collections. The following query is a variant of the prior group-by query example. Instead of listing all messages for every sender location, the query retrieves a list of the top three reply messages with smallest message-ids for each sender location.

##### Example

    SELECT loc AS location, 
      (
            SELECT ELEMENT m.message 
            FROM x AS m
            WHERE  m.`in-response-to` > 0
            ORDER BY m.`message-id`
            LIMIT 3
      ) AS replies
    FROM FacebookMessages AS x
    GROUP BY x.`author-id` AS loc;

It returns:

    [ 
      { "replies": [ " dislike iphone its touch-screen is horrible", " can't stand at&t the network is horrible:(", " like verizon the 3G is awesome:)" ], "location": 1 }, 
      { "replies": [ " like samsung the plan is amazing", " like t-mobile its platform is mind-blowing" ], "location": 2 }
    ]

## <a id="DDL_and_DML_Statements">3. DDL and DML Statements</a> <font size="4"><a href="#toc">[Back to TOC]</a></font>

    Statement ::= ( SingleStatement ( ";" )? )* <EOF>
    SingleStatement ::= DatabaseDeclaration
                      | FunctionDeclaration
                      | CreateStatement
                      | DropStatement
                      | LoadStatement
                      | SetStatement
                      | InsertStatement
                      | DeleteStatement
                      | Query ";"

In addition to queries, SQL++ supports a variety of statements for data definition and manipulation purposes as well as controlling the context to be used in evaluating SQL++ expressions. SQL++ supports record-level ACID transactions that begin and terminate implicitly for each record inserted, deleted, or searched while a given SQL++ statement is being executed.

This section details the statements supported in the SQL++ language.

### Declarations

    DatbaseDeclaration ::= "USE" Identifier

The world of data in an AsterixDB cluster is organized into data namespaces called databases. To set the default database for a series of statements, the use statement is provided.

As an example, the following statement sets the default database to be TinySocial.

##### Example

    USE TinySocial;

The set statement in SQL++ is used to control aspects of the expression evalation context for queries.

    SetStatement ::= "SET" Identifier StringLiteral

As an example, the following set statements request that Jaccard similarity with a similarity threshold 0.6 be used for set similarity matching when the ~= operator is used in a query expression.

##### Example

    SET simfunction "jaccard";
    SET simthreshold "0.6f";

When writing a complex SQL++ query, it can sometimes be helpful to define one or more
auxilliary functions that each address a sub-piece of the overall query. The declare function statement supports the creation of such helper functions.

    FunctionDeclaration  ::= "DECLARE" "FUNCTION" Identifier ParameterList "{" Expression "}"
    ParameterList        ::= "(" ( <VARIABLE> ( "," <VARIABLE> )* )? ")"

The following is a very simple example of a temporary SQL++ function definition.

##### Example

    DECLARE FUNCTION add(a, b) {
      a + b
    };

### Lifecycle Management Statements

    CreateStatement ::= "CREATE" ( DatabaseSpecification
                                 | TypeSpecification
                                 | TableSpecification
                                 | IndexSpecification
                                 | FunctionSpecification )

    QualifiedName       ::= Identifier ( "." Identifier )?
    DoubleQualifiedName ::= Identifier "." Identifier ( "." Identifier )?

The create statement in SQL++ is used for creating persistent artifacts in the context of database. It can be used to create new databases, datatypes, tables, indexes, and user-defined SQL++ functions.

#### Databases

    DatabaseSpecification ::= "DATABASE" Identifier IfNotExists ( "WITH" "FORMAT" StringLiteral )?

The create database statement is used to create new databases. To ease the authoring of reusable SQL++ scripts, its optional IfNotExists clause allows creation to be requested either unconditionally or only if the the database does not already exist. If this clause is absent, an error will be returned if the specified database already exists. The `WITH FORMAT` clause is a placeholder for future functionality that can safely be ignored.

The following example creates a database named TinySocial.

##### Example

    CREATE DATABASE TinySocial;

#### Types

    TypeSpecification    ::= "TYPE" FunctionOrTypeName IfNotExists "AS" TypeExpr
    FunctionOrTypeName   ::= QualifiedName
    IfNotExists          ::= ( "IF" "NOT" "EXISTS" )?
    TypeExpr             ::= RecordTypeDef | TypeReference | OrderedListTypeDef | UnorderedListTypeDef
    RecordTypeDef        ::= ( "CLOSED" | "OPEN" )? "{" ( RecordField ( "," RecordField )* )? "}"
    RecordField          ::= Identifier ":" ( TypeExpr ) ( "?" )?
    NestedField          ::= Identifier ( "." Identifier )*
    IndexField           ::= NestedField ( ":" TypeReference )?
    TypeReference        ::= Identifier
    OrderedListTypeDef   ::= "[" ( TypeExpr ) "]"
    UnorderedListTypeDef ::= "{{" ( TypeExpr ) "}}"

The create type statement is used to create a new named ADM datatype. This type can then be used to create tables or utilized when defining one or more other ADM datatypes. Much more information about the Asterix Data Model (ADM) is available in the [data model reference guide](datamodel.html) to ADM. A new type can be a record type, a renaming of another type, an ordered list type, or an unordered list type. A record type can be defined as being either open or closed. Instances of a closed record type are not permitted to contain fields other than those specified in the create type statement. Instances of an open record type may carry additional fields, and open is the default for a new type (if neither option is specified).

The following example creates a new ADM record type called FacebookUser type. Since it is closed, its instances will contain only what is specified in the type definition. The first four fields are traditional typed name/value pairs. The friend-ids field is an unordered list of 32-bit integers. The employment field is an ordered list of instances of another named record type, EmploymentType.

##### Example

    CREATE TYPE FacebookUserType AS CLOSED {
      "id" :         int32,
      "alias" :      string,
      "name" :       string,
      "user-since" : datetime,
      "friend-ids" : {{ int32 }},
      "employment" : [ EmploymentType ]
    }

The next example creates a new ADM record type called FbUserType. Note that the type of the id field is UUID. You need to use this field type if you want to have this field be an autogenerated-PK field. Refer to the Tables section later for more details.

##### Example

    CREATE TYPE FbUserType AS CLOSED {
      "id" :         uuid,
      "alias" :      string,
      "name" :       string
    }

#### Tables

    TableSpecification ::= "INTERNAL"? "table" QualifiedName "(" QualifiedName ")" IfNotExists PrimaryKey ( "ON" Identifier )? ( "HINTS" Properties )? ( "USING" "COMPACTION" "POLICY" CompactionPolicy ( Configuration )? )? ( "WITH FILTER ON" Identifier )? | "EXTERNAL" "TABLE" QualifiedName "(" QualifiedName ")" IfNotExists "USING" AdapterName Configuration ( "HINTS" Properties )? ( "USING" "COMPACTION" "POLICY" CompactionPolicy ( Configuration )? )?
    AdapterName          ::= Identifier
    Configuration        ::= "(" ( KeyValuePair ( "," KeyValuePair )* )? ")"
    KeyValuePair         ::= "(" StringLiteral "=" StringLiteral ")"
    Properties           ::= ( "(" Property ( "," Property )* ")" )?
    Property             ::= Identifier "=" ( StringLiteral | IntegerLiteral )
    FunctionSignature    ::= FunctionOrTypeName "@" IntegerLiteral
    PrimaryKey           ::= "PRIMARY" "KEY" NestedField ( "," NestedField )* ( "AUTOGENERATED")?
    CompactionPolicy     ::= Identifier
    PrimaryKey           ::= "PRIMARY" "KEY" Identifier ( "," Identifier )* ( "AUTOGENERATED ")?

The create table statement is used to create a new table. Tables are named, unordered collections of ADM record instances; they are where data lives persistently and are the targets for queries in AsterixDB. Tables are typed, and AsterixDB will ensure that their contents conform to their type definitions. An Internal table (the default) is a table that is stored in and managed by AsterixDB. It must have a specified unique primary key that can be used to partition data across nodes of an AsterixDB cluster. The primary key is also used in secondary indexes to uniquely identify the indexed primary data records. Random primary key (UUID) values can be auto-generated by declaring the field to be UUID and putting "AUTOGENERATED" after the "PRIMARY KEY" identifier. In this case, values for the auto-generated PK field should not be provided by the user since it will be auto-generated by AsterixDB. Optionally, a filter can be created on a field to further optimize range queries with predicates on the filter's field. (Refer to [Filter-Based LSM Index Acceleration](filters.html) for more information about filters.)

An External table is stored outside of AsterixDB (currently tables in HDFS or on the local filesystem(s) of the cluster's nodes are supported). External table support allows SQL++ queries to treat external data as though it were stored in AsterixDB, making it possible to query "legacy" file data (e.g., Hive data) without having to physically import it into AsterixDB. For an external table, an appropriate adapter must be selected to handle the nature of the desired external data. (See the [guide to external data](externaldata.html) for more information on the available adapters.)

When creating a table, it is possible to choose a merge policy that controls which of the underlaying LSM storage components to be merged.  Currently, AsterixDB provides four different merge policies that can be configured per table: no-merge, constant, prefix, and correlated-prefix. The no-merge policy simply never merges disk components. While the constant policy merges disk components when the number of components reaches some constant number k, which can be configured by the user. The prefix policy relies on component sizes and the number of components to decide which components to merge. Specifically, it works by first trying to identify the smallest ordered (oldest to newest) sequence of components such that the sequence does not contain a single component that exceeds some threshold size M and that either the sum of the component's sizes exceeds M or the number of components in the sequence exceeds another threshold C. If such a sequence of components exists, then each of the components in the sequence are merged together to form a single component. Finally, the correlated-prefix is similar to the prefix policy but it delegates the decision of merging the disk components of all the indexes in a table to the primary index. When the policy decides that the primary index needs to be merged (using the same decision criteria as for the prefix policy), then it will issue successive merge requests on behalf of all other indexes associated with the same table. The default policy for AsterixDB is the prefix policy except when there is a filter on a table, where the preferred policy for filters is the correlated-prefix.

The following example creates an internal table for storing FacefookUserType records.
It specifies that their id field is their primary key.

##### Example
    CREATE INTERNAL TABLE FacebookUsers(FacebookUserType) PRIMARY KEY id;

The following example creates an internal table for storing FbUserType records. It specifies that their id field is their primary key. It also specifies that the id field is an auto-generated field, meaning that a randomly generated UUID value will be assigned to each record by the system. (A user should therefore not proivde a value for this field.) Note that the id field should be UUID.

##### Example
    CREATE INTERNAL TABLE FbMsgs(FbUserType) PRIMARY KEY id AUTOGENERATED;

The next example creates an external table for storing LineitemType records. The choice of the `hdfs` adapter means that its data will reside in HDFS. The create statement provides parameters used by the hdfs adapter: the URL and path needed to locate the data in HDFS and a description of the data format.

##### Example
    CREATE EXTERNAL TABLE Lineitem('LineitemType) USING hdfs (
      ("hdfs"="hdfs://HOST:PORT"),
      ("path"="HDFS_PATH"),
      ("input-format"="text-input-format"),
      ("format"="delimited-text"),
      ("delimiter"="|"));

#### Indices

    IndexSpecification ::= "INDEX" Identifier IfNotExists "ON" QualifiedName "(" ( IndexField ) ( "," IndexField )* ")" ( "type" IndexType )? ( "enforced" )?
    IndexType          ::= "BTREE"
                         | "RTREE"
                         | "KEYWORD"
                         | "NGRAM" "(" IntegerLiteral ")"

The create index statement creates a secondary index on one or more fields of a specified table. Supported index types include `BTREE` for totally ordered datatypes, `RTREE` for spatial data, and `KEYWORD` and `NGRAM` for textual (string) data. An index can be created on a nested field (or fields) by providing a valid path expression as an index field identifier.

An index field is not required to be part of the datatype associated with a table if that datatype is declared as open and the field's type is provided along with its type and the `ENFORCED` keyword is specified in the end of index definition. `ENFORCING` an open field will introduce a check that will make sure that the actual type of an indexed field (if the field exists in the record) always matches this specified (open) field type.

The following example creates a btree index called fbAuthorIdx on the author-id field of the FacebookMessages table. This index can be useful for accelerating exact-match queries, range search queries, and joins involving the author-id field.

##### Example

    CREATE INDEX fbAuthorIdx ON FacebookMessages(author-id) TYPE BTREE;

The following example creates an open btree index called fbSendTimeIdx on the open send-time field of the FacebookMessages table having datetime type. This index can be useful for accelerating exact-match queries, range search queries, and joins involving the send-time field.

##### Example

    CREATE INDEX fbSendTimeIdx ON FacebookMessages(send-time:datetime) TYPE BTREE ENFORCED;

The following example creates a btree index called twUserScrNameIdx on the screen-name field, which is a nested field of the user field in the TweetMessages table. This index can be useful for accelerating exact-match queries, range search queries, and joins involving the screen-name field.

##### Example

    CREATE INDEX twUserScrNameIdx ON TweetMessages(user.screen-name) TYPE BTREE;

The following example creates an rtree index called fbSenderLocIdx on the sender-location field of the FacebookMessages table. This index can be useful for accelerating queries that use the [`spatial-intersect` function](functions.html#spatial-intersect) in a predicate involving the sender-location field.

##### Example

    CREATE INDEX fbSenderLocIndex ON FacebookMessages("sender-location") TYPE RTREE;

The following example creates a 3-gram index called fbUserIdx on the name field of the FacebookUsers table. This index can be used to accelerate some similarity or substring maching queries on the name field. For details refer to the [document on similarity queries](similarity.html#NGram_Index).

##### Example

    CREATE INDEX fbUserIdx ON FacebookUsers(name) TYPE NGRAM(3);

The following example creates a keyword index called fbMessageIdx on the message field of the FacebookMessages table. This keyword index can be used to optimize queries with token-based similarity predicates on the message field. For details refer to the [document on similarity queries](similarity.html#Keyword_Index).

##### Example

    CREATE INDEX fbMessageIdx ON FacebookMessages(message) TYPE KEYWORD;

#### Functions

The create function statement creates a named function that can then be used and reused in SQL++ queries. The body of a function can be any SQL++ expression involving the function's parameters.

    FunctionSpecification ::= "FUNCTION" FunctionOrTypeName IfNotExists ParameterList "{" Expression "}"

The following is a very simple example of a create function statement. It differs from the declare function example shown previously in that it results in a function that is persistently registered by name in the specified database.

##### Example

    CREATE FUNCTION add(a, b) {
      a + b
    };

#### Removal

    DropStatement       ::= "DROP" ( "DATABASE" Identifier IfExists
                                   | "TYPE" FunctionOrTypeName IfExists
                                   | "TABLE" QualifiedName IfExists
                                   | "INDEX" DoubleQualifiedName IfExists
                                   | "FUNCTION" FunctionSignature IfExists )
    IfExists            ::= ( "IF" "EXISTS" )?

The drop statement in SQL++ is the inverse of the create statement. It can be used to drop databases, datatypes, tables, indexes, and functions.

The following examples illustrate uses of the drop statement.

##### Example

    DROP TABLE FacebookUsers IF EXISTS;

    DROP INDEX fbSenderLocIndex;

    DROP TYPE FacebookUserType;

    DROP DATABASE TinySocial;

    DROP FUNCTION add;

### Import/Export Statements

    LoadStatement  ::= "LOAD" "TABLE" QualifiedName "USING" AdapterName Configuration ( "PRE-SORTED" )?

The load statement is used to initially populate a table via bulk loading of data from an external file. An appropriate adapter must be selected to handle the nature of the desired external data. The load statement accepts the same adapters and the same parameters as external tables. (See the [guide to external data](externaldata.html) for more information on the available adapters.) If a table has an auto-generated primary key field, a file to be imported should not include that field in it.

The following example shows how to bulk load the FacebookUsers table from an external file containing data that has been prepared in ADM format.

##### Example

    LOAD TABLE FacebookUsers USING localfs
    (("path"="localhost:///Users/zuck/AsterixDB/load/fbu.adm"),("format"="adm"));

### Modification Statements

#### Insert

    InsertStatement ::= "INSERT" "INTO" "TABLE" QualifiedName Query

The SQL++ insert statement is used to insert data into a table. The data to be inserted comes from an SQL++ query expression. The expression can be as simple as a constant expression, or in general it can be any legal SQL++ query. Inserts in AsterixDB are processed transactionally, with the scope of each insert transaction being the insertion of a single object plus its affiliated secondary index entries (if any). If the query part of an insert returns a single object, then the insert statement itself will be a single, atomic transaction. If the query part returns multiple objects, then each object inserted will be handled independently as a tranaction. If a table has an auto-generated primary key field, an insert statement should not include a value for that field in it. (The system will automatically extend the provided record with this additional field and a corresponding value.)

The following example illustrates a query-based insertion.

##### Example

    INSERT INTO TABLE UsersCopy (FROM FacebookUsers user SELECT ELEMENT user)

#### Delete

    DeleteStatement ::= "DELETE" Variable "FROM" "TABLE" QualifiedName ( "WHERE" Expression )?

The SQL++ delete statement is used to delete data from a target table. The data to be deleted is identified by a boolean expression involving the variable bound to the target table in the delete statement. Deletes in AsterixDB are processed transactionally, with the scope of each delete transaction being the deletion of a single object plus its affiliated secondary index entries (if any). If the boolean expression for a delete identifies a single object, then the delete statement itself will be a single, atomic transaction. If the expression identifies multiple objects, then each object deleted will be handled independently as a transaction.

The following example illustrates a single-object deletion.

##### Example

    DELETE user FROM TABLE FacebookUsers WHERE user.id = 8;

We close this guide to SQL++ with one final example of a query expression.

##### Example

    FROM {{ "great", "brilliant", "awesome" }} AS praise
    SELECT ELEMENT string-concat(["AsterixDB is ", praise])
