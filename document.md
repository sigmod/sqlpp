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
  * [Path expressions](#Path_expressions)
  * [Operator expressions](#Operator_expressions)
    * [Arithmetic operators](#Arithmetic_operators)
    * [Collection operators](#Collection_operators)
    * [Comparison operators](#Comparison_operators)
    * [Logical operators](#Logical_operators)
  * [Case expressions](#Case_expressions)
  * [Quantified expressions](#Quantified_expressions)
* [3. Queries](#Queries)
  * [SELECT statements](#SELECT_statements)
  * [SELECT clauses](#Select_clauses)
    * [Select element/value/raw](#Select_element)
    * [SQL-style select](#SQL_select)
    * [Select *](#Select_star)
    * [Select distinct](#Select_distinct)
    * [Unnamed projections](#Unnamed_projections)
    * [Abbreviatory field access expressions](#Abbreviatory_field_access_expressions)
  * [UNNEST clauses](#Unnest_clauses)
    * [Inner unnests](#Inner_unnests)
    * [Left outer unnests](#Left_outer_unnests)
    * [Expressing joins using unnests](#Expressing_joins_using_unnests)
  * [FROM clauses](#From_clauses)
    * [Binding expressions](#Binding_expressions)
    * [Multiple from terms](#Multiple_from_terms)
    * [Expressing joins using from terms](#Expressing_joins_using_from_terms)
    * [Implicit binding variables](#Implicit_binding_variables)
  * [JOIN clauses](#Join_clauses)
    * [Inner joins](#Inner_joins)
    * [Left outer joins](#Left_outer_joins)
  * [GROUP BY clauses](#Group_By_clauses)
    * [Group variables](#Group_variables)
    * [Implicit group key variables](#Implicit_group_key_variables)
    * [Implicit group variables](#Implicit_group_variables)
    * [Aggregation functions](#Aggregation_functions)
    * [SQL-92 aggregation functions](#SQL-92_aggregation_functions)
    * [SQL-92 compliant GROUP BY aggregations](#SQL-92_compliant_gby)
    * [Column aliases](#Column_alias)
  * [WHERE clauases and HAVING clauses](#Where_having_clauses)
  * [ORDER BY clauses](#Order_By_clauses)
  * [LIMIT clauses](#Limit_clauses)
  * [WITH clauses](#With_clauses)
  * [LET clauses](#Let_clauses)
  * [MISSING in query results](#Missing_in_query_results)
  * [SQL++ Vs. SQL-92](#Vs_SQL-92)
* [4. DDL and DML statements](#DDL_and_DML_statements)
  * [Declarations](#Declarations)
  * [Lifecycle management statements](#Lifecycle_management_statements)
    * [Databases](#Databases)
    * [Tables](#Tables)
    * [Types](#Types)
    * [Functions](#Functions)
  * [Modification statements](#Modification_statements)
    * [Inserts](#Inserts)
    * [Deletes](#Deletes)

# <a id="Introduction">1. Introduction</a><font size="4">

This document is intended as a reference guide to the full syntax and semantics of the SQL++ Query Language, a SQL-inspired language for working with Apache AsterixDB. SQL++ has much in common with SQL, but there are also differences due to the data model that the language is designed to serve. (SQL was designed in the 1970's for interacting with the flat, schema-ified world of relational databases, while SQL++ is designed for the nested, schema-less/schema-optional world of modern NoSQL systems.) In particular, SQL++ in the context of AsterixDB is aimed at working with the Asterix Data Model (ADM), which (roughly speaking) is a data model aimed at a superset of JSON.

New AsterixDB users are encouraged to read and work through the (friendlier) guide "AsterixDB 101: An ADM and SQL++ Primer" before attempting to make use of this document. In addition, readers are advised to read and understand the Asterix Data Model (ADM) reference guide since a basic understanding of ADM concepts is a prerequisite to understanding SQL++. In what follows, we detail the features of the SQL++ language in a grammar-guided manner: we list and briefly explain each of the productions in the SQL++ grammar, offering examples for clarity in cases where doing so seems needed or helpful.

# <a id="Expressions">2. Expressions

    Expression ::= OperatorExpression | CaseExpression | QuantifiedExpression

SQL++ is a highly composable expression language. Each SQL++ expression returns zero or more Asterix Data Model (ADM) instances. There are three major kinds of expressions in SQL++. At the topmost level, a SQL++ expression can be an OperatorExpression (similar to a mathematical expression), an ConditionalExpression (to choose between alternative values), or a QuantifiedExpression (which yields a boolean value). Each will be detailed as we explore the full SQL++ grammar.

## <a id="Primary_expressions">Primary Expressions

    PrimaryExpr ::= Literal
                  | VariableReference
                  | ParenthesizedExpression
                  | FunctionCallExpression
                  | Constructor

The most basic building block for any SQL++ expression is PrimaryExpression. This can be a simple literal (constant) value, a reference to a query variable that is in scope, a parenthesized expression, a function call, or a newly constructed instance of the Asterix Data Model (such as a newly constructed ADM record or list of ADM instances).

### <a id="Literals">Literals

    Literal        ::= StringLiteral
                       | IntegerLiteral
                       | FloatLiteral
                       | DoubleLiteral
                       | <NULL>
                       | <MISSING>
                       | <TRUE>
                       | <FALSE>
    StringLiteral  ::= "\'" (<ESCAPE_APOS> | ~["\'"])* "\'"
                       | "\"" (<ESCAPE_APOS> | ~["\'"])* "\""
    <ESCAPE_APOS>  ::= "\\\'"
    IntegerLiteral ::= <DIGITS>
    <DIGITS>       ::= ["0" - "9"]+
    FloatLiteral   ::= <DIGITS> ( "f" | "F" )
                     | <DIGITS> ( "." <DIGITS> ( "f" | "F" ) )?
                     | "." <DIGITS> ( "f" | "F" )
    DoubleLiteral  ::= <DIGITS>
                     | <DIGITS> ( "." <DIGITS> )?
                     | "." <DIGITS>

> MC: I tentatively deleted the following unused ESCAPE_QUOTE definition: <ESCAPE_QUOT>  ::= "\\\""
> 		<ESCAPE_QUOT>  ::= "\\\""
> Also, I moved the DelimitedIdentifier down further per TW's suggestion.

Literals (constants) in SQL++ can be strings, integers, floating point values, double values, boolean constants, or special constant values like `NULL` and `MISSING`. The `NULL` value is like a `NULL` in SQL; it is used to represent an unknown field value. The specialy value `MISSING` is only meaningful in the context of SQL++ field accesses; it occurs when the accessed field simply does not exist at all in a record being accessed.

The following are some simple examples of SQL++ literals.

#### Examples

    'a string'
    "test string"
    42

Different from standard SQL, double quotes play the same role as single quotes and may be used for string literals in SQL++.

### <a id="Variable_references">Variable References

    VariableReference ::= <IDENTIFIER>|<DelimitedIdentifier>
    <IDENTIFIER>  ::= <LETTER> (<LETTER> | <DIGIT> | "_" | "$")*
    <LETTER>    ::= ["A" - "Z", "a" - "z"]
    DelimitedIdentifier   ::= "\`" (<ESCAPE_APOS> | ~["\'"])* "\`"

A variable in SQL++ can be bound to any legal ADM value. A variable reference refers to the value to which an in-scope variable is bound. (E.g., a variable binding may originate from one of the `FROM`, `WITH` or `LET` clauses of a `SELECT` statement or from an input parameter in the context of a function body.) Backticks, e.g., \`id\`, are used for delimited identifiers. Delimiting is needed when a variable's desired name clashes with a SQL++ keyword or includes characters not allowed in regular identifiers.

#### Examples

    tweet
    id
    `SELECT`
    `my-function`

### <a id="Parenthesized_expressions">Parenthesized expressions

    ParenthesizedExpression ::= "(" Expression ")" | Subquery

An expression can be parenthesized to control the precedence order or otherwise clarify a query. In SQL++, for composability, a subquery is also an parenthesized expression.

The following expression evaluates to the value 2.

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

A major feature of SQL++ is its ability to construct new ADM data instances. This is accomplished using its constructors for each of the major ADM complex object structures, namely lists (ordered or unordered) and records. Ordered lists are like JSON arrays, while unordered lists have multiset (bag) semantics. Records are built from attributes that are field-name/field-value pairs, again like JSON. (See the AsterixDB Data Model document for more details on each.)

The following examples illustrate how to construct a new ordered list with 3 items, a new record with 2 fields, and a new unordered list with 4 items, respectively. List elements can be homogeneous (as in the first example), which is the common case, or they may be heterogeneous (as in the third example). The data values and field name values used to construct lists and records in constructors are all simply SQL++ expressions. Thus, the list elements, field names, and field values used in constructors can be simple literals or they can come from query variable references or even arbitrarily complex SQL++ expressions (subqueries).

#### Examples

    [ 'a', 'b', 'c' ]

    {
      'project name': 'AsterixDB',
      'project members': [ 'vinayakb', 'dtabass', 'chenli', 'tsotras' ]
    }

    {{ 42, "forty-two!", { "rank": "Captain", "name": "America" }, 3.14159 }}

### <a id="Path_expressions">Path expressions

    PathExpression  ::= PrimaryExpression ( Field | Index )*
    Field           ::= "." Identifier
    Index           ::= "[" ( Expression | "?" ) "]"

Components of complex types in ADM are accessed via path expressions. Path access can be applied to the result of a SQL++ expression that yields an instance of  a complex type, e.g., a record or list instance. For records, path access is based on field names. For ordered lists, path access is based on (zero-based) array-style indexing. SQL++ also supports an "I'm feeling lucky" style index accessor, [?], for selecting an arbitrary element from an ordered list. Attempts to access non-existent fields or out-of-bound list elements produce the special value `MISSING`.

The following examples illustrate field access for a record, index-based element access for an ordered list, and also a composition thereof.

#### Examples

    ({"name": "MyABCs", "list": [ "a", "b", "c"]}).list

    (["a", "b", "c"])[2]

    ({"name": "MyABCs", "list": [ "a", "b", "c"]}).list[2]

### <a id="Operator_expressions">Operator expressions
    
Operators perform a specific operation on the input values or expressions. The syntax of an operator expression is as follows:

    OperatorExpression ::= PathExpression
                           | Operator OperatorExpression
                           | OperatorExpression Operator (OperatorExpression)?

SQL++ provides a full set of operators that you can use within its statements. Here are the categories of operators:

* [Arithmetic operators](#Arithmetic_operators), to perform basic mathematical operations;
* [Collection operators](#Collection_operators), to evaluate expressions on collections or objects;
* [Comparison operators](#Comparison_operators), to compare two expressions;
* [Logical Operators](#Logical_operators), to combine operators using Boolean logic.

The following table summarizes the precedence order (from higher to lower) of the major unary and binary operators:

| Operator                                                                    | Operation |
|-----------------------------------------------------------------------------|-----------|
| EXISTS, NOT EXISTS                                                          |  collection emptiness testing |
| ^                                                                           |  exponentiation  |
| *, /                                                                        |  multiplication, division |
| +, -                                                                        |  addition, subtraction  |
| IS NULL, IS NOT NULL, IS MISSING, IS NOT MISSING, <br>IS UNKNOWN, IS NOT UNKNOWN| unknown value comparison |
| =, !=, <, >, <=, >=, LIKE, NOT LIKE, IN, NOT IN                             | comparison  |
| NOT                                                                         | logical negation |
| AND                                                                         | conjunction |
| OR                                                                          | disjunction |

> MC: I messed with the table above, which was way (way) off, so someone (Yingyi :-)) should check my work!

### <a id="Arithmetic_operators">Arithmetic operators
Arithemtic operators are used to exponentiate, negate, add, subtract, multiply, and divide numeric values.
 
| Operator |  Purpose                                                                       | Example    |
|----------|--------------------------------------------------------------------------------|------------|
| +, -     |  As unary operators, they denote a <br>positive or negative expression         | SELECT VALUE -1; |
| +, -     |  As binary operators, they add or subtract                                     | SELECT VALUE 1 + 2; |
| *, /     |  Multiply, divide                                                              | SELECT VALUE 4 / 2.0; |
| ^        |  Exponentiation                                                                | SELECT VALUE 2^3;       |

### <a id="Collection_operators">Collection operators
Collection operators are used for membership tests (IN, NOT IN) or empty collection tests (EXISTS, NOT EXISTS).

| Operator   |  Purpose                                     | Example    |
|------------|----------------------------------------------|------------|
| IN         |  Membership test                             | SELECT * FROM ChirpMessages cm <br>WHERE cm.user.lang IN ["en", "de"]; |
| NOT IN     |  Non-membership test                         | SELECT * FROM ChirpMessages cm <br>WHERE cm.user.lang NOT IN ["en"]; |
| EXISTS     |  Check whether a collection is not empty     | SELECT * FROM ChirpMessages cm <br>WHERE EXISTS cm.referredTopics; |
| NOT EXISTS |  Check whether a collection is empty         | SELECT * FROM ChirpMessages cm <br>WHERE NOT EXISTS cm.referredTopics; |

### <a id="Comparison_operators">Comparison operators
Comparison operators are used to compare values. The comparison operators fall into one of two sub-categories: missing value comparisons and regular value comparisons. SQL++ (and JSON) has two ways of representing missing information in a record - the presence of the field with a NULL for its value (as in SQL), and the absence of the field (which JSON permits). For example, the first of the following records represents Jack, whose friend is Jill. In the other examples, Jake is friendless a la SQL, with a friend field that is NULL, while Joe is friendless in a more natural (for JSON) way, i.e., by not having a friend field.

#### Examples
{"name": "Jack", "friend": "Jill"}

{"name": "Jake", "friend": NULL}

{"name": "Joe"}

The following table enumerates all of SQL++'s comparison operators. 

| Operator       |  Purpose                                   | Example    |
|----------------|--------------------------------------------|------------|
| IS NULL        |  Test if a value is NULL                       | SELECT * FROM ChirpMessages cm <br>WHERE cm.user.name IS NULL; |
| IS NOT NULL    |  Test if a value is not NULL                   | SELECT * FROM ChirpMessages cm <br>WHERE cm.user.name IS NOT NULL; | 
| IS MISSING     |  Test if a value is MISSING                    | SELECT * FROM ChirpMessages cm <br>WHERE cm.user.name IS MISSING; |
| IS NOT MISSING |  Test if a value is not MISSING                | SELECT * FROM ChirpMessages cm <br>WHERE cm.user.name IS NOT MISSING;|
| IS UNKNOWN     |  Test if a value is NULL or MISSING            | SELECT * FROM ChirpMessages cm <br>WHERE cm.user.name IS UNKNOWN; | 
| IS NOT UNKNOWN |  Test if a value is neither NULL nor MISSING   | SELECT * FROM ChirpMessages cm <br>WHERE cm.user.name IS NOT UNKNOWN;| 
| =              |  Equality test                                 | SELECT * FROM ChirpMessages cm <br>WHERE cm.chirpId=10; |
| !=             |  Inequality test                               | SELECT * FROM ChirpMessages cm <br>WHERE cm.chirpId!=10;|
| <              |  Less than                                     | SELECT * FROM ChirpMessages cm <br>WHERE cm.chirpId<10; |
| >              |  Greater than                                  | SELECT * FROM ChirpMessages cm <br>WHERE cm.chirpId>10; |
| <=             |  Less than or equal to                         | SELECT * FROM ChirpMessages cm <br>WHERE cm.chirpId<=10; |
| >=             |  Greater than or equal to                      | SELECT * FROM ChirpMessages cm <br>WHERE cm.chirpId>=10; |
| LIKE           |  Test if the left side matches a<br> pattern defined on the right<br> side; in the pattern,  "%" matches  <br>any string while "_" matches <br> any character. | SELECT * FROM ChirpMessages cm <br>WHERE cm.user.name LIKE "%Giesen%";|
| NOT LIKE       |  Test if the left side does not <br>match a pattern defined on the right<br> side; in the pattern,  "%" matches <br>any string while "_" matches <br> any character. | SELECT * FROM ChirpMessages cm <br>WHERE cm.user.name NOT LIKE "%Giesen%";| 

The following table summarizes how the missing value comparison operators work.

| Operator | Non-NULL/Non-MISSING value | NULL | MISSING |
|----------|----------------|------|---------|
| IS NULL  | FALSE | TRUE | FALSE |
| IS NOT NULL | TRUE | FALSE | TRUE |
| IS MISSING  | FALSE | FALSE | TRUE |
| IS NOT MISSING | TRUE | TRUE | FALSE |
| IS UNKNOWN | FALSE | TRUE | TRUE |
| IS NOT UNKNOWN | TRUE | FALSE | FALSE|

### <a id="Logical_operators">Logical operators
Logical operators perform logical `NOT`, `AND`, and `OR` operations over Boolean values (`TRUE` and `FALSE`) plus `NULL` and `MISSING`.

| Operator |  Purpose                                   | Example    |
|----------|-----------------------------------------------------------------------------|------------|
| NOT      |  Returns true if the following condition is false, otherwise returns false  | SELECT VALUE NOT TRUE;  |
| AND      |  Returns true if both branches are true, otherwise returns false            | SELECT VALUE TRUE AND FALSE; | 
| OR       |  Returns true if one branch is true, otherwise returns false                | SELECT VALUE FALSE OR FALSE; |

The following table is the truth table for `AND` and `OR`.

| A  | B  | A AND B  | A OR B |
|----|----|----------|--------|
| TRUE | TRUE | TRUE | TRUE |
| TRUE | FALSE | FALSE | TRUE |
| TRUE | NULL | NULL | TRUE |
| TRUE | MISSING | MISSING | TRUE |
| FALSE | FALSE | FALSE | FALSE |
| FALSE | NULL | FALSE | NULL |
| FALSE | MISSING | FALSE | MISSING |
| NULL | NULL | NULL | NULL |
| NULL | MISSING | MISSING | NULL |
| MISSING | MISSING | MISSING | MISSING |

The following table demonstrates the results of `NOT` on all possible inputs.

| A  | NOT A |
|----|----|
| TRUE | FALSE |
| FALSE | TRUE |
| NULL | NULL |
| MISSING | MISSING | 

### <a id="Case_expressions">Case expressions

    CaseExpression ::= SimpleCaseExpression | SearchedCaseExpression
    SimpleCaseExpression ::= <CASE> Expression ( <WHEN> Expression <THEN> Expression )+ ( <ELSE> Expression )? <END>
    SearchedCaseExpression ::= <CASE> ( <WHEN> Expression <THEN> Expression )+ ( <ELSE> Expression )? <END>

In a simple `CASE` expression, the query evaluator searches for the first `WHEN` ... `THEN` pair in which the `WHEN` expression is equal to the expression following `CASE` and returns the expression following `THEN`. If none of the `WHEN` ... `THEN` pairs meet this condition, and an `ELSE` branch exists, it returns the `ELSE` expression. Otherwise, `NULL` is returned.

In a searched CASE expression, the query evaluator searches from left to right until it finds a `WHEN` expression that is evaluated to `TRUE`, and then returns its corresponding `THEN` expression. If no condition is found to be `TRUE`, and an `ELSE` branch exists, it returns the `ELSE` expression. Otherwise, it returns `NULL`.

The following example illustrates the form of a case expression.
#### Example

    CASE (2 < 3) WHEN true THEN "yes" ELSE "no" END

### <a id="Quantified_expressions">Quantified expressions

    QuantifiedExpression ::= ( <SOME> | <EVERY> ) Variable <IN> Expression ( "," Variable "in" Expression )*
                             <SATISFIES> Expression

Quantified expressions are used for expressing existential or universal predicates involving the elements of a collection.

The following pair of examples illustrate the use of a quantified expression to test that every (or some) element in the set [1, 2, 3] of integers is less than three. The first example yields `FALSE` and second example yields `TRUE`.

It is useful to note that if the set were instead the empty set, the first expression would yield `TRUE` ("every" value in an empty set satisfies the condition) while the second expression would yield `FALSE` (since there isn't "some" value, as there are no values in the set, that satisfies the condition).

##### Examples

    EVERY x IN [ 1, 2, 3 ] SATISFIES x < 3
    SOME x IN [ 1, 2, 3 ] SATISFIES x < 3


# <a id="Queries">3. Queries</a>

A SQL++ query can be any legal SQL++ expression or `SELECT` statement. A SQL++ query always ends with a semicolon.

    Query ::= (Expression | SelectStatement) ";"

##  <a id="SELECT_statements">SELECT statements

The following shows the (rich) grammar for the `SELECT` statement in SQL++.

> TW: Should we replace SelectElement with SelectValue? MC: Yes, and done below.

    SelectStatement    ::= ( WithClause )?
                           SelectSetOperation (OrderbyClause )? ( LimitClause )?
    SelectSetOperation ::= SelectBlock (<UNION> <ALL> ( SelectBlock | Subquery ) )*
    Subquery           ::= "(" SelectStatement ")"
    
    SelectBlock        ::= SelectClause
                           ( FromClause ( WithClause )?)?
                           ( WhereClause )?
                           ( GroupbyClause ( LetClause )? ( HavingClause )? )?
                           |
                           FromClause ( WithClause )?
                           ( WhereClause )?
                           ( GroupbyClause ( WithClause )? ( HavingClause )? )?
                           SelectClause

    SelectClause       ::= <SELECT> ( <ALL> | <DISTINCT> )? ( SelectRegular | SelectValue )
    SelectRegular      ::= Projection ( "," Projection )*
    SelectValue      ::= ( <VALUE> | <ELEMENT> | <RAW> ) Expression
    Projection         ::= ( Expression ( <AS> )? Identifier | "*" )
  
    FromClause         ::= <FROM> FromTerm ( "," FromTerm )*
    FromTerm           ::= Expression (( <AS> )? Variable)? ( <AT> Variable )?
                           ( ( JoinType )? ( JoinClause | UnnestClause ) )*

    JoinClause         ::= <JOIN> Expression (( <AS> )? Variable)? (<AT> Variable)? <ON> Expression
    UnnestClause       ::= ( <UNNEST> | <CORRELATE> | <FLATTEN> ) Expression
                           ( <AS> )? Variable ( <AT> Variable )?
    JoinType           ::= ( <INNER> | <LEFT> ( <OUTER> )? )

    WithClause         ::= <WITH> WithElement ( "," WithElement )*
    LetClause          ::= (<LET> | <LETTING>) LetElement ( "," LetElement )*
    LetElement         ::= Variable "=" Expression
    WithElement        ::= Variable <AS> Expression
  
    WhereClause        ::= <WHERE> Expression
    
    GroupbyClause      ::= <GROUP> <BY> ( Expression ( (<AS>)? Variable )? ( "," Expression ( (<AS>)? Variable )? )*
                           ( <GROUP> <AS> Variable
                             ("(" Variable <AS> VariableReference ("," Variable <AS> VariableReference )* ")")?
                           )?
    HavingClause       ::= <HAVING> Expression
    
    OrderbyClause      ::= <ORDER> <BY> Expression ( <ASC> | <DESC> )? ( "," Expression ( <ASC> | <DESC> )? )*
    LimitClause        ::= <LIMIT> Expression ( <OFFSET> Expression )?

In this section, we will make use of two stored collections of records (datasets in ADM parlance), `GleambookUsers` and `GleambookMessages`, in a series of running examples to explain `SELECT` queries. The contents of the example collections are as follows:

`GleambookUsers` collection:

    {"id":1,"alias":"Margarita","name":"MargaritaStoddard","nickname":"Mags","userSince":datetime("2012-08-20T10:10:00"),"friendIds":{{2,3,6,10}},"employment":[{"organizationName":"Codetechno","start-date":date("2006-08-06")},{"organizationName":"geomedia","start-date":date("2010-06-17"),"end-date":date("2010-01-26")}],"gender":"F"}
    {"id":2,"alias":"Isbel","name":"IsbelDull","nickname":"Izzy","userSince":datetime("2011-01-22T10:10:00"),"friendIds":{{1,4}},"employment":[{"organizationName":"Hexviafind","startDate":date("2010-04-27")}]}
    {"id":3,"alias":"Emory","name":"EmoryUnk","userSince":datetime("2012-07-10T10:10:00"),"friendIds":{{1,5,8,9}},"employment":[{"organizationName":"geomedia","startDate":date("2010-06-17"),"endDate":date("2010-01-26")}]}

`GleambookMessages` collection:

    {"messageId":2,"authorId":1,"inResponseTo":4,"senderLocation":point("41.66,80.87"),"message":" dislike iphone its touch-screen is horrible"}
    {"messageId":3,"authorId":2,"inResponseTo":4,"senderLocation":point("48.09,81.01"),"message":" like samsung the plan is amazing"}
    {"messageId":4,"authorId":1,"inResponseTo":2,"senderLocation":point("37.73,97.04"),"message":" can't stand at&t the network is horrible:("}
    {"messageId":6,"authorId":2,"inResponseTo":1,"senderLocation":point("31.5,75.56"),"message":" like t-mobile its platform is mind-blowing"}
    {"messageId":8,"authorId":1,"inResponseTo":11,"senderLocation":point("40.33,80.87"),"message":" like verizon the 3G is awesome:)"}
    {"messageId":10,"authorId":1,"inResponseTo":12,"senderLocation":point("42.5,70.01"),"message":" can't stand motorola the touch-screen is terrible"}
    {"messageId":11,"authorId":1,"inResponseTo":1,"senderLocation":point("38.97,77.49"),"message":" can't stand at&t its plan is terrible"}

## <a id="Select_clauses">SELECT Clause
The SQL++ `SELECT` clause always returns a collection value as its result (even if the result is empty or a singleton).

### <a id="Select_element">SELECT VALUE Clause
The `SELECT VALUE` clause in SQL++ returns a collection that contains the results of evaluating the `VALUE` expression, with one evaluation being performed per "binding tuple" (i.e., per `FROM` clause item) satisfying the statement's selection criteria.
For historical reasons SQL++ also allows the keywords `ELEMENT` or `RAW` to be used in place of `VALUE` (not recommended).
The following example shows a query that selects one user from the GleambookUsers collection.

#### Example

    SELECT VALUE user
    FROM GleambookUsers user
    WHERE user.id = 1;

This query returns:

    [
      { "id": 1, "alias": "Margarita", "name": "MargaritaStoddard", "userSince": datetime("2012-08-20T10:10:00.000Z"), "friendIds": {{ 2, 3, 6, 10 }}, "employment": [ { "organizationName": "Codetechno", "startDate": date("2006-08-06") }, { "organizationName": "geomedia", "startDate": date("2010-06-17"), "endDate": date("2010-01-26") } ], "nickname": "Mags", "gender": "F" }

    ]

### <a id="SQL_select">SQL-style SELECT
In SQL++, the traditional SQL-style `SELECT` syntax is also supported.
This syntax can also be reformulated in a `SELECT VALUE` based manner in SQL++.
(E.g., `SELECT expA AS fldA, expB AS fldB` is syntactic sugar for `SELECT VALUE { 'fldA': expA, 'fldB': expB }`.)

#### Example
    SELECT user.alias user_alias, user.name user_name
    FROM GleambookUsers user
    WHERE user.id = 1;
    
Returns:

    [
      {"user_alias":"Margarita","user_name":"MargaritaStoddard"}
    ]

### <a id="Select_star">SELECT *      
In SQL++, `SELECT *` returns a record with a nested field for each input tuple. Each field has as its field name the name of a binding variable generated by either the `FROM` clause or `GROUP BY` clause in the current enclosing `SELECT` statement, and its field is the value of that binding variable.

#### Example

    SELECT *
    FROM GleambookUsers user;

Since `user` is the only binding variable generated in the `FROM` clause, this query returns:

    [
      { "user": { "id": 1, "alias": "Margarita", "name": "MargaritaStoddard", "userSince": datetime("2012-08-20T10:10:00.000Z"), "friendIds": {{ 2, 3, 6, 10 }}, "employment": [ { "organizationName": "Codetechno", "startDate": date("2006-08-06") }, { "organizationName": "geomedia", "startDate": date("2010-06-17"), "endDate": date("2010-01-26") } ], "nickname": "Mags", "gender": "F" } }
      { "user": { "id": 2, "alias": "Isbel", "name": "IsbelDull", "userSince": datetime("2011-01-22T10:10:00.000Z"), "friendIds": {{ 1, 4 }}, "employment": [ { "organizationName": "Hexviafind", "startDate": date("2010-04-27") } ], "nickname": "Izzy" } }
      { "user": { "id": 3, "alias": "Emory", "name": "EmoryUnk", "userSince": datetime("2012-07-10T10:10:00.000Z"), "friendIds": {{ 1, 5, 8, 9 }}, "employment": [ { "organizationName": "geomedia", "startDate": date("2010-06-17"), "endDate": date("2010-01-26") } ] } }
    ]

### <a id="Select_distinct">SELECT DISTINCT
SQL++'s `DISTINCT` keyword is used to eliminate duplicate items in results. The following example shows how it works.

#### Example

    SELECT DISTINCT * FROM [1, 2, 2, 3] AS foo;

This query returns:

    [ 
      { "foo": 1 },
      { "foo": 2 },
      { "foo": 3 }
    ]

#### Example

    SELECT DISTINCT VALUE foo FROM [1, 2, 2, 3] AS foo;

This version of the query returns:

    [ 1, 2, 3 ]

### <a id="Unnamed_projections">Unnamed projections
Similar to standard SQL, SQL++ supports unnamed projections (a.k.a, unnamed `SELECT` clause items), for which names are generated.
Name generation has three cases:

  * If a projection expression is a variable reference expression, its generated name is the name of the variable.
  * If a projection expression is a field access expression, its generated name is the last identifier in the expression.
  * For all other cases, the query processor will generate a unique name.

#### Example

    SELECT substr(user.name, 10), user.alias
    FROM GleambookUsers user
    WHERE user.id = 1;
    
This query outputs:

    [ 
      { "$1": "Stoddard", "alias": "Margarita" }
    ]

In the result, `$1` is the generated name for `substr(user.name, 1)`, while `alias` is the generated name for `user.alias`.

### <a id="Abbreviatory_field_access_expressions">Abbreviated Field Access Expressions
As in standard SQL, SQL++ field access expressions can be abbreviated (not recommended) when there is no ambiguity. In the next example, the variable `user` is the only possible variable reference for fields `id`, `name` and `alias` and thus could be omitted in the query.

#### Example

    SELECT substr(name, 10) AS lname, alias
    FROM GleambookUsers user
    WHERE id = 1;

Outputs:

    [ 
      { "lname": "Stoddard", "alias": "Margarita" }
    ]

## <a id="Unnest_clauses">UNNEST Clause
For each of its input tuples, the `UNNEST` clause flattens a collection-valued expression into individual items, producing multiple tuples, each of which is one of the expression's original input tuples augmented with a flattened item from its collection.

### <a id="Inner_unnests">Inner UNNEST
The following example is a query that retrieves the names of the organizations that a selected user has worked for. It uses the `UNNEST` clause to unnest the nested collection `employment` in the user's record.

#### Example

    SELECT u.id AS userId, e.organizationName AS orgName
    FROM GleambookUsers u
    UNNEST u.employment e
    WHERE u.id = 1;

This query returns:

    [
      { "userId": 1, "orgName": "Codetechno" }
      { "userId": 1, "orgName": "geomedia" }
    ]

Note that `UNNEST` has SQL's inner join semantics --- that is, if a user has no employment history, no tuple corresponding to that user will be emitted in the result.

### <a id="Left_outer_unnests">Left outer UNNEST
As an alternative, the `LEFT OUTER UNNEST` clause offers SQL's left outer join semantics. For example, no collection-valued field named `hobbies` exists in the record for the user whose id is 1, but the following query's result still includes user 1.

#### Example

    SELECT u.id AS userId, h.hobbyName AS hobby
    FROM GleambookUsers u
    LEFT OUTER UNNEST u.hobbies h
    WHERE u.id = 1;

Returns:

    [
      { "userId": 1 }
    ]

Note that if `u.hobbies` is an empty collection or leads to a `MISSING` (as above) or `NULL` value for a given input tuple, there is no corresponding binding value for variable `h` for an input tuple. A `MISSING` value will be generated for `h` so that the input tuple can still be propagated.

### <a id="Expressing_joins_using_unnests">Expressing joins using UNNEST
The SQL++ `UNNEST` clause is similar to SQL's `JOIN` clause except that it allows its right argument to be correlated to its left argument, as in the examples above --- i.e., think "correlated cross-product".
The next example shows this via a query that joins two data sets, GleambookUsers and GleambookMessages, returning user/message pairs. The results contain one record per pair, with result records containing the user's name and an entire message. The query can be thought of as saying "for each Gleambook user, unnest the `GleambookMessages` collection and filter the output with the condition `message.authorId = user.id`".

#### Example

    SELECT u.name AS uname, m.message AS message
    FROM GleambookUsers u
    UNNEST GleambookMessages m
    WHERE m.authorId = u.id;

This returns:

    [
      { "uname": "MargaritaStoddard", "message": " can't stand at&t its plan is terrible" }
      { "uname": "MargaritaStoddard", "message": " dislike iphone its touch-screen is horrible" }
      { "uname": "MargaritaStoddard", "message": " can't stand at&t the network is horrible:(" }
      { "uname": "MargaritaStoddard", "message": " like verizon the 3G is awesome:)" }
      { "uname": "MargaritaStoddard", "message": " can't stand motorola the touch-screen is terrible" }
      { "uname": "IsbelDull", "message": " like t-mobile its platform is mind-blowing" }
      { "uname": "IsbelDull", "message": " like samsung the plan is amazing" }
    ]  

Similarly, the above query can also be expressed as the `UNNEST`ing of a correlated SQL++ subquery:

#### Example

    SELECT u.name AS uname, m.message AS message
    FROM GleambookUsers u
    UNNEST (
        SELECT VALUE msg
        FROM GleambookMessages msg
        WHERE msg.authorId = u.id
    ) AS m;

## <a id="From_clauses">FROM clauses
A `FROM` clause is used for enumerating (i.e., conceptually iterating over) the contents of collections, as in SQL.

### <a id="Binding_expressions">Binding expressions
In SQL++, in addition to stored collections, a `FROM` clause can iterate over any intermediate collection returned by a valid SQL++ expression.

#### Example

    SELECT VALUE foo
    FROM [1, 2, 2, 3] AS foo
    WHERE foo > 2;

Returns:

    [
      3
    ]

### <a id="Multiple_from_terms">Multiple FROM terms
SQL++ permits correlations among `FROM` terms. Specifically, a `FROM` binding expression can refer to variables defined to its left in the given `FROM` clause. Thus, the first unnesting example above could also be expressed as follows:

#### Example

    SELECT u.id AS userId, e.organizationName AS orgName
    FROM GleambookUsers u, u.employment e
    WHERE u.id = 1;


### <a id="Expressing_joins_using_from_terms">Expressing joins using FROM terms
Similarly, the join intentions of the other `UNNEST`-based join examples above could be expressed as:

#### Example

    SELECT u.name AS uname, m.message AS message
    FROM GleambookUsers u, GleambookMessages m
    WHERE m.authorId = u.id;

#### Example

    SELECT u.name AS uname, m.message AS message
    FROM GleambookUsers u,
      (
        SELECT VALUE msg
        FROM GleambookMessages msg
        WHERE msg.authorId = u.id
      ) AS m;

Note that the first alternative is one of the SQL-92 approaches to expressing a join.

### <a id="Implicit_binding_variables">Implicit binding variables

Similar to standard SQL, SQL++ supports implicit `FROM` binding variables (i.e., aliases), for which a binding variable is generated. SQL++ variable generation falls into three cases:

  * If the binding expression is a variable reference expression, the generated variable's name will be the name of the referenced variable itself.
  * If the binding expression is a field access expression, the generated variable's name will be the last identifier in the expression.
  * For all other cases, a compilation error will be raised.

The next two examples show queries that do not provide binding variables in their `FROM` clauses.

#### Example

    SELECT GleambookUsers.name, GleambookMessages.message
    FROM GleambookUsers, GleambookMessages
    WHERE GleambookMessages.authorId = GleambookUsers.id;

Returns:

    [
      { "name": "MargaritaStoddard", "message": " can't stand at&t its plan is terrible" }
      { "name": "MargaritaStoddard", "message": " dislike iphone its touch-screen is horrible" }
      { "name": "MargaritaStoddard", "message": " can't stand at&t the network is horrible:(" }
      { "name": "MargaritaStoddard", "message": " like verizon the 3G is awesome:)" }
      { "name": "MargaritaStoddard", "message": " can't stand motorola the touch-screen is terrible" }
      { "name": "IsbelDull", "message": " like t-mobile its platform is mind-blowing" }
      { "name": "IsbelDull", "message": " like samsung the plan is amazing" }
    ]

#### Example

    SELECT GleambookUsers.name, GleambookMessages.message
    FROM GleambookUsers, 
      (
        SELECT VALUE GleambookMessages
        FROM GleambookMessages
        WHERE GleambookMessages.authorId = GleambookUsers.id
      );

Returns:

    Error: Need an alias for the enclosed expression:
    (select element $GleambookMessages
        from $GleambookMessages as $GleambookMessages
        where ($GleambookMessages.authorId = $GleambookUsers.id)
    )

## <a id="Join_clauses">JOIN clauses
The join clause in SQL++ supports both inner joins and left outer joins from standard SQL.

### <a id="Inner_joins">Inner joins
Using a `JOIN` clause, the inner join intent from the preceeding examples can also be expressed as follows:

#### Example

    SELECT u.name AS uname, m.message AS message
    FROM GleambookUsers u JOIN GleambookMessages m ON m.authorId = u.id;

### <a id="Left_outer_joins">Left outer joins
SQL++ supports SQL's notion of left outer join. The following query is an example:

    SELECT u.name AS uname, m.message AS message
    FROM GleambookUsers u LEFT OUTER JOIN GleambookMessages m ON m.authorId = u.id;

Returns:

    [ 
      { "uname": "MargaritaStoddard", "message": " can't stand at&t its plan is terrible" }
      { "uname": "MargaritaStoddard", "message": " dislike iphone its touch-screen is horrible" }
      { "uname": "MargaritaStoddard", "message": " can't stand at&t the network is horrible:(" }
      { "uname": "MargaritaStoddard", "message": " like verizon the 3G is awesome:)" }
      { "uname": "MargaritaStoddard", "message": " can't stand motorola the touch-screen is terrible" }
      { "uname": "IsbelDull", "message": " like t-mobile its platform is mind-blowing" }
      { "uname": "IsbelDull", "message": " like samsung the plan is amazing" }
      { "uname": "EmoryUnk" }
    ]
   
For non-matching left-side tuples, SQL++ produces `MISSING` values for the right-side binding variables; that is why the last record in the above result doesn't have a `message` field. Note that this is slightly different from standard SQL, which instead would fill in `NULL` values for the right-side fields. The reason for this difference is that, for non-matches in its join results, SQL++ views fields from the right-side as being "not there" (a.k.a. `MISSING`) instead of as being "there but unknown" (i.e., `NULL`).

The left-outer join query can also be expressed using `LEFT OUTER UNNEST`:

    SELECT u.name AS uname, m.message AS message
    FROM GleambookUsers u
    LEFT OUTER UNNEST (
        SELECT VALUE message
        FROM GleambookMessages message
        WHERE message.authorId = u.id
      ) m;

In general, in SQL++, SQL-style join queries can also be expressed by `UNNEST` clauses and left outer join queries can be expressed by `LEFT OUTER UNNESTs`.

## <a id="Group_By_clauses">GROUP BY clauses
The SQL++ `GROUP BY` clause generalizes standard SQL's grouping and aggregation semantics, but it also retains backward compatibility with the standard (relational) SQL `GROUP BY` and aggregation features.

### <a id="Group_variables">Group variables
In a `GROUP BY` clause, in addition to the binding variable(s) defined for the grouping key(s), SQL++ allows a user to define a *group variable* by using the clause's `GROUP AS` extension to denote the resulting group.
After grouping, then, the query's in-scope variables include the grouping key's binding variables as well as this group variable which will be bound to one collection value for each group. This per-group collection value will be a set of nested records in which each field of the record is the result of a renamed variable defined in parentheses following the group variable's name. The `GROUP AS` syntax is as follows:

    <GROUP> <AS> Variable ("(" Variable <AS> VariableReference ("," Variable <AS> VariableReference )* ")")?

#### Example

    SELECT *
    FROM GleambookMessages message
    GROUP BY message.authorId AS uid GROUP AS msgs(message AS msg);

This first example query returns:

    [
       { "uid": 1, "msgs": [ { "msg": { "messageId": 8, "authorId": 1, "inResponseTo": 11, "senderLocation": point("40.33,80.87"), "message": " like verizon the 3G is awesome:)" } }, 
                             { "msg": { "messageId": 10, "authorId": 1, "inResponseTo": 12, "senderLocation": point("42.5,70.01"), "message": " can't stand motorola the touch-screen is terrible" } }, 
                             { "msg": { "messageId": 11, "authorId": 1, "inResponseTo": 1, "senderLocation": point("38.97,77.49"), "message": " can't stand at&t its plan is terrible" } }, 
                             { "msg": { "messageId": 2, "authorId": 1, "inResponseTo": 4, "senderLocation": point("41.66,80.87"), "message": " dislike iphone its touch-screen is horrible" } }, 
                             { "msg": { "messageId": 4, "authorId": 1, "inResponseTo": 2, "senderLocation": point("37.73,97.04"), "message": " can't stand at&t the network is horrible:(" } } ] }
       { "uid": 2, "msgs": [ { "msg": { "messageId": 6, "authorId": 2, "inResponseTo": 1, "senderLocation": point("31.5,75.56"), "message": " like t-mobile its platform is mind-blowing" } }, 
                             { "msg": { "messageId": 3, "authorId": 2, "inResponseTo": 4, "senderLocation": point("48.09,81.01"), "message": " like samsung the plan is amazing" } } ] }
    ]

As we can see from the above query result, each group in the example query's output has an associated group
variable value called `msgs` that appears in the `SELECT *`'s result.
This variable contains a collection of records associated with the group; each of the group's `message` values
appears in the `msg` field of the records in the `msgs` collection.

The group variable in SQL++ makes more complex, composable, nested subqueries over a group possible, which is
important given the more complex data model of SQL++ (relative to SQL).
As a simple example of this, as we really just want the messages associated with each user, we might wish to avoid
the "extra wrapping" of each message as the `msg` field of a record.
(That wrapping is useful in more complex cases, but is essentially just in the way here.)
We can use a subquery in the `SELECT` clase to tunnel through the extra nesting and produce the desired result.

#### Example

    SELECT uid, (SELECT VALUE m.msg FROM msgs m) AS msgs
    FROM GleambookMessages message
    GROUP BY message.authorId AS uid GROUP AS msgs(message AS msg);

This variant of the example query returns:

       { "uid": 1, "msgs": [ { "messageId": 8, "authorId": 1, "inResponseTo": 11, "senderLocation": point("40.33,80.87"), "message": " like verizon the 3G is awesome:)" },
                             { "messageId": 10, "authorId": 1, "inResponseTo": 12, "senderLocation": point("42.5,70.01"), "message": " can't stand motorola the touch-screen is terrible" },
                             { "messageId": 11, "authorId": 1, "inResponseTo": 1, "senderLocation": point("38.97,77.49"), "message": " can't stand at&t its plan is terrible" }, 
                             { "messageId": 2, "authorId": 1, "inResponseTo": 4, "senderLocation": point("41.66,80.87"), "message": " dislike iphone its touch-screen is horrible" }, 
                             { "messageId": 4, "authorId": 1, "inResponseTo": 2, "senderLocation": point("37.73,97.04"), "message": " can't stand at&t the network is horrible:(" } ] }
       { "uid": 2, "msgs": [ { "messageId": 6, "authorId": 2, "inResponseTo": 1, "senderLocation": point("31.5,75.56"), "message": " like t-mobile its platform is mind-blowing" }, 
                             { "messageId": 3, "authorId": 2, "inResponseTo": 4, "senderLocation": point("48.09,81.01"), "message": " like samsung the plan is amazing" } ] }

Because this is a fairly common case, a third variant with output identical to the second variant is also possible:

#### Example

    SELECT uid, msg AS msgs
    FROM GleambookMessages message
    GROUP BY message.authorId AS uid GROUP AS msgs(message AS msg);

This variant of the query exploits a bit of SQL-style "syntactic sugar" that SQL++ offers to shorten some user queries.
In particular, in the `SELECT` list, the reference to the `GROUP` variable field `msg` -- because it references a field of the group variable -- is allowed but is "pluralized". As a result, the `msg` reference in the `SELECT` list is
implicitly rewritten into the second variant's `SELECT VALUE` subquery.

The next example shows a more interesting case involving the use of a subquery in the `SELECT` list.
Here the subquery further processes the groups.

#### Example

    SELECT uid,
           (SELECT VALUE m.msg
            FROM msgs m
            WHERE m.msg.message LIKE '% like%'
            ORDER BY m.msg.messageId
            LIMIT 2) AS msgs
    FROM GleambookMessages message
    GROUP BY message.authorId AS uid GROUP AS msgs(message AS msg);

This example query returns:

    [
      { "uid": 1, "msgs": [ { "messageId": 8, "authorId": 1, "inResponseTo": 11, "senderLocation": point("40.33,80.87"), "message": " like verizon the 3G is awesome:)" } ] }
      { "uid": 2, "msgs": [ { "messageId": 3, "authorId": 2, "inResponseTo": 4, "senderLocation": point("48.09,81.01"), "message": " like samsung the plan is amazing" },
                            { "messageId": 6, "authorId": 2, "inResponseTo": 1, "senderLocation": point("31.5,75.56"), "message": " like t-mobile its platform is mind-blowing" } ] }
    ]

### <a id="Implicit_group_key_variables">Implicit grouping key variables
In the SQL++ syntax, providing named binding variables for `GROUP BY` key expressions is optional.
If a grouping key is missing a user-provided binding variable, the underlying compiler will generate one.
Automatic grouping key variable naming falls into three cases in SQL++, much like the treatment of unnamed projections:

  * If the grouping key expression is a variable reference expression, the generated variable gets the same name as the referred variable;
  * If the grouping key expression is a field access expression, the generated variable gets the same name as the last identifier in the expression;
  * For all other cases, the compiler generates a unique variable (but the user query is unable to refer to this generated variable).

The next example illustrates a query that doesn't provide binding variables for its grouping key expressions.

#### Example

    SELECT authorId,
           (SELECT VALUE m.msg
            FROM msgs m
            WHERE m.msg.message LIKE '% like%'
            ORDER BY m.msg.messageId
            LIMIT 2) AS msgs
    FROM GleambookMessages message
    GROUP BY message.authorId GROUP AS msgs(message AS msg);

This query returns:

    [
      { "authorId": 1, "msgs": [ { "messageId": 8, "authorId": 1, "inResponseTo": 11, "senderLocation": point("40.33,80.87"), "message": " like verizon the 3G is awesome:)" } ] }
      { "authorId": 2, "msgs": [ { "messageId": 3, "authorId": 2, "inResponseTo": 4, "senderLocation": point("48.09,81.01"), "message": " like samsung the plan is amazing" },
                                 { "messageId": 6, "authorId": 2, "inResponseTo": 1, "senderLocation": point("31.5,75.56"), "message": " like t-mobile its platform is mind-blowing" } ] }
    ]

Based on the three variable generation rules, the generated variable for the grouping key expression `message.authorId`
is `authorId` (which is how it is referred to in the example's `SELECT` clause).

### <a id="Implicit_group_variables">Implicit group variables
The group variable itself is also optional in SQL++'s `GROUP BY` syntax.
If a user's query does not declare the name and structure of the group variable using `GROUP AS`,
the query compiler will generate a unique group variable whose fields include all of the
binding variables defined in the `FROM` clause of the current enclosing `SELECT` statement.
(In this case the user's query will not be able to refer to the generated group variable.)

#### Example

    SELECT uid,
           (SELECT m.message
            FROM message m
            WHERE m.message LIKE '% like%'
            ORDER BY m.messageId
            LIMIT 2) AS msgs
    FROM GleambookMessages message
    GROUP BY message.authorId AS uid;

This query returns:

    [
      { "uid": 1, "msgs": [ { "message": " like verizon the 3G is awesome:)" } ] }
      { "uid": 2, "msgs": [ { "message": " like samsung the plan is amazing" },
                            { "message": " like t-mobile its platform is mind-blowing" } ] }
    ]

Note that in the query above, in principle, `message` is not an in-scope variable in the `SELECT` clause.
However, the query above is a syntactically-sugared simplification of the following query and it is thus
legal, executable, and returns the same result:

    SELECT uid,
           (SELECT m.message
            FROM (SELECT VALUE grp.message FROM `$1` AS grp) AS m
            WHERE m.message LIKE '% like%'
            ORDER BY m.messageId
            LIMIT 2) AS msgs
    FROM GleambookMessages message
    GROUP BY message.authorId AS uid GROUP AS `$1` (message AS message);

### <a id="Aggregation_functions">Aggregation functions
In traditional SQL, which doesn't support nested data, grouping always also involves the use of aggregation
compute properties of the groups (e.g., the average number of messages per user rather than the actual set
of messages per user).
Each aggregation function in SQL++ takes a collection (e.g., the group of messages) as its input and produces
a scalar value as its output.
These aggregation functions, being truly functional in nature (unlike in SQL), can be used anywhere in a
query where an expression is allowed.
The following table catalogs the SQL++ built-in aggregation functions and also indicates how each one handles
`NULL`/`MISSING` values in the input collection or a completely empty input collection:

| Function       | NULL         | MISSING      | Empty Collection |
|----------------|--------------|--------------|------------------|
| COLL_COUNT     | counted      | counted      | 0                |
| COLL_SUM       | returns NULL | returns NULL | returns NULL     |
| COLL_MAX       | returns NULL | returns NULL | returns NULL     |
| COLL_MIN       | returns NULL | returns NULL | returns NULL     |
| COLL_AVG       | returns NULL | returns NULL | returns NULL     |
| COLL_SQL-COUNT | not counted  | not counted  | 0                |
| COLL_SQL-SUM   | ignores NULL | ignores NULL | returns NULL     |
| COLL_SQL-MAX   | ignores NULL | ignores NULL | returns NULL     |
| COLL_SQL-MIN   | ignores NULL | ignores NULL | returns NULL     |
| COLL_SQL-AVG   | ignores NULL | ignores NULL | returns NULL     |

Notice that SQL++ has twice as many functions listed above as there are aggregate functions in SQL-92.
This is because SQL++ offers two versions of each -- one that handles `UNKNOWN` values in a semantically
strict fashion, where unknown values in the input result in unknown values in the output -- and one that
handles them in the ad hoc "just ignore the unknown values" fashion that the SQL standard chose to adopt.

#### Example

    COLL_AVG(
        (
          SELECT VALUE len(friendIds) FROM GleambookUsers
        )
    );

This example returns:

    3.3333333333333335

#### Example

    SELECT uid AS uid, COLL_COUNT(grp) AS msgCnt
    FROM GleambookMessages message
    GROUP BY message.authorId AS uid GROUP AS grp(message AS msg);
    
This query returns:

    [
      { "uid": 1, "msgCnt": 5 }
      { "uid": 2, "msgCnt": 2 }
    ]

Notice how the query forms groups where each group involves a message author and their messages.
(SQL cannot do this because the grouped intermediate result is non-1NF in nature.)
The query then uses the collection aggregate function `COLL_COUNT` to get the cardinality of each
group of messages.
    
### <a id="SQL-92_aggregation_functions">SQL-92 aggregation functions
For compatibility with the traditional SQL aggregation functions, SQL++ also offers SQL-92's
aggregation function symbols (`COUNT`, `SUM`, `MAX`, `MIN`, and `AVG`) as supported syntactic sugar.
The SQL++ compiler rewrites queries that utilize these function symbols into SQL++ queries that only
use the SQL++ collection aggregate functions. The following example uses the SQL-92 syntax approach
to compute a result that is identical to that of the more explicit SQL++ example above:

#### Example

    SELECT uid, COUNT(msg) AS msgCnt
    FROM GleambookMessages msg
    GROUP BY msg.authorId AS uid;

It is important to realize that `COUNT` is actually **not** a SQL++ built-in aggregation function.
Rather, the `COUNT` query above is using a special "sugared" function symbol that the SQL++ compiler
will rewrite as follows:

    SELECT uid AS uid, `COLL_SQL-COUNT`( (SELECT g.msg FROM `$1` as g) ) AS msgCnt
    FROM GleambookMessages msg
    GROUP BY msg.authorId AS uid GROUP AS `$1`(msg AS msg);

> TW: We really need to do something about `COLL_SQL-COUNT`.
> MC: You mean about its name? And inconsistent dashing? I agree...!  :-)
> Also, do we need to say anything about the (mandatory) double parens here?

The same sort of rewritings apply to the function symbols `SUM`, `MAX`, `MIN`, and `AVG`.
In contrast to the SQL++ collection aggregate functions, these special SQL-92 function symbols
can only be used in the same way they are in standard SQL (i.e., with the same restrictions).

### <a id="SQL-92_compliant_gby">SQL-92 compliant GROUP BY aggregations
SQL++ provides full support for SQL-92 `GROUP BY` aggregation queries.
The following query is such an example:

#### Example

    SELECT msg.authorId, COUNT(msg)
    FROM GleambookMessages msg
    GROUP BY msg.authorId;
    
This query outputs:

    [ 
      { "authorId": 1, "$1": 5 },
      { "authorId": 2, "$1": 2 }
    ]

In principle, a `msg` reference in the query's `SELECT` clause would be "sugarized" as a collection
(as described in [Implicit group variables](#Implicit_group_variables)).
However, since the SELECT expression `msg.authorId` is syntactically identical to a GROUP BY key expression,
it will be internally replaced by the generated group key variable.
The following is the equivalent rewritten query that will be generated by the compiler for the query above:

    SELECT authorId AS authorId, COLL_COUNT( (SELECT g.msg FROM `$1` AS g) )
    FROM GleambookMessages msg
    GROUP BY msg.authorId AS authorId GROUP AS `$1`(msg AS msg);

### <a id="Column_aliases">Column aliases
SQL++ also allows column aliases to be used as `GROUP BY` keys or `ORDER BY` keys.

#### Example

    SELECT msg.authorId AS aid, COUNT(msg)
    FROM GleambookMessages msg
    GROUP BY aid;

This query returns:

    [ 
      { "aid": 1, "$1": 5 }
      { "aid": 2, "$1": 2 }
    ]

## <a id="Where_having_clauses">WHERE clauses and HAVING clauses
Both `WHERE` clauses and `HAVING` clauses are used to filter input data based on a condition expression.
Only tuples for which the condition expression evaluates to `TRUE` are propagated.
Note that if the condition expression evaluates to `NULL` or `MISSING` the input tuple will be disgarded.

## <a id="Order_By_clauses">ORDER BY clauses
The `ORDER BY` clause is used to globally sort data in either ascending order (i.e., `ASC`) or descending order (i.e., `DESC`).
During ordering, `MISSING` and `NULL` are treated as being smaller than any other value if they are encountered
in the ordering key(s). `MISSING` is treated as smaller than `NULL` if both occur in the data being sorted.
The following example returns all `GleambookUsers` ordered by their friend numbers.

#### Example

      SELECT VALUE user
      FROM GleambookUsers AS user
      ORDER BY len(user.friendIds) DESC;

This query returns:

    [
      { "id": 1, "alias": "Margarita", "name": "MargaritaStoddard", "userSince": datetime("2012-08-20T10:10:00.000Z"), "friendIds": {{ 2, 3, 6, 10 }}, "employment": [ { "organizationName": "Codetechno", "startDate": date("2006-08-06") }, { "organizationName": "geomedia", "startDate": date("2010-06-17"), "endDate": date("2010-01-26") } ], "nickname": "Mags", "gender": "F" }
      { "id": 3, "alias": "Emory", "name": "EmoryUnk", "userSince": datetime("2012-07-10T10:10:00.000Z"), "friendIds": {{ 1, 5, 8, 9 }}, "employment": [ { "organizationName": "geomedia", "startDate": date("2010-06-17"), "endDate": date("2010-01-26") } ] }
      { "id": 2, "alias": "Isbel", "name": "IsbelDull", "userSince": datetime("2011-01-22T10:10:00.000Z"), "friendIds": {{ 1, 4 }}, "employment": [ { "organizationName": "Hexviafind", "startDate": date("2010-04-27") } ], "nickname": "Izzy" }
    ]

## <a id="Limit_clauses">LIMIT clauses
The `LIMIT` clause is used to limit the result set to a specified constant size.
The use of the `LIMIT` clause is illustrated in the next example.

#### Example

      SELECT VALUE user
      FROM GleambookUsers AS user
      ORDER BY len(user.friendIds) DESC
      LIMIT 1;

This query returns:

    [
      { "id": 1, "alias": "Margarita", "name": "MargaritaStoddard", "userSince": datetime("2012-08-20T10:10:00.000Z"), "friendIds": {{ 2, 3, 6, 10 }}, "employment": [ { "organizationName": "Codetechno", "startDate": date("2006-08-06") }, { "organizationName": "geomedia", "startDate": date("2010-06-17"), "endDate": date("2010-01-26") } ], "nickname": "Mags", "gender": "F" }
    ]

## <a id="With_clauses">WITH clauses
As in standard SQL, `WITH` clauses are available to improve the modularity of a query.
The next query shows an example.

#### Example

    WITH avgFriendCount AS ( 
      SELECT VALUE AVG(LEN(user.friendIds))
      FROM GleambookUsers AS user
    )[0]
    SELECT VALUE user
    FROM GleambookUsers user
    WHERE LEN(user.friendIds) > avgFriendCount;

This query returns:

    [
      { "id": 1, "alias": "Margarita", "name": "MargaritaStoddard", "userSince": datetime("2012-08-20T10:10:00.000Z"), "friendIds": {{ 2, 3, 6, 10 }}, "employment": [ { "organizationName": "Codetechno", "startDate": date("2006-08-06") }, { "organizationName": "geomedia", "startDate": date("2010-06-17"), "endDate": date("2010-01-26") } ], "nickname": "Mags", "gender": "F" }
      { "id": 3, "alias": "Emory", "name": "EmoryUnk", "userSince": datetime("2012-07-10T10:10:00.000Z"), "friendIds": {{ 1, 5, 8, 9 }}, "employment": [ { "organizationName": "geomedia", "startDate": date("2010-06-17"), "endDate": date("2010-01-26") } ] }
    ]

The query is equivalent to the following, more complex, inlined form of the query:

    SELECT *
    FROM GleambookUsers user
    WHERE LEN(user.friendIds) >
        ( SELECT VALUE AVG(LEN(user.friendIds))
          FROM GleambookUsers AS user
        ) [0];

WITH can be particularly useful when a value needs to be used several times in a query.

Before proceeding further, notice that both  the WITH query and its equivalent inlined variant
include the syntax "[0]" -- this is due to a noteworthy difference between SQL++ and SQL-92.
In SQL-92, whenever a scalar value is expected and it is being produced by a query expression,
the SQL-92 query processor will evaluate the expression, check that there is only one row and column
in the result at runtime, and then coerce the one-row/one-column tabular result into a scalar value.
SQL++, being designed to deal with nested data and schema-less data, does not (and should not) do this.
Collection-valued data is perfectly legal in most SQL++ contexts, and its data is schema-less,
so a query processor rarely knows exactly what to expect where and such automatic conversion is often
not desirable. Thus, in the queries above, the use of "[0]" extracts the first (i.e., 0th) element of
a list-valued query expression's result; this is needed above, even though the result is a list of one
element, to "de-listify" the list and obtain the desired scalar for the comparison.

## <a id="Subqueries">Subqueries
In SQL++, an arbitrary subquery can appear anywhere that an expression can appear.
Unlike SQL-92, as was just alluded to, the subqueries in a SELECT list or a boolean predicate need
not return singleton, single-column relations.
Instead, they may return arbitrary collections.
For example, the following query is a variant of the prior group-by query examples;
it retrieves a list of up to two "dislike" messages per user.

##### Example

    SELECT uid,
           (SELECT VALUE m.msg
            FROM msgs m
            WHERE m.msg.message LIKE '%dislike%'
            ORDER BY m.msg.messageId
            LIMIT 2) AS msgs
    FROM GleambookMessages message
    GROUP BY message.authorId AS uid GROUP AS msgs(message AS msg);

For our sample data set, this query returns:

    [
      { "uid": 1, "msgs": [ { "messageId": 2, "authorId": 1, "inResponseTo": 4, "senderLocation": point("41.66,80.87"), "message": " dislike iphone its touch-screen is horrible" } ] }
      { "uid": 2, "msgs": [  ] }
    ]

Note that a subquery, like a top-level `SELECT` statment, always returns a collection -- regardless of where
within a query the subquery occurs -- and again, its result is never automatically cast into a scalar.

## <a id="Vs_SQL-92">SQL++ vs. SQL-92
The following matrix is a quick "key differences cheat sheet" for SQL++ and SQL-92.

| Feature |  SQL++ | SQL-92 |
|----------|--------|--------|
| SELECT * | Returns nested records. | Returns flattened concatenated records. |
| Subquery | Returns a collection.  | The returned collection of records is cast into a scalar value if the subquery appears in a SELECT list or on one side of a comparison or as input to a function. |
| Left outer join |  Fills in `MISSING` for non-matches.  |   Fills in `NULL`(s) for non-matches.    |
| String literal | Double quotes or single quotes. | Single quotes only. |
| Delimited identifiers | Backticks. | Double quotes. |

For things beyond the cheat sheet, SQL++ is SQL-92 compliant.
Morever, SQL++ offers the following additional features beyond SQL-92 (hence the "++" in its name):

  * Fully composable and functional: A subquery can iterate over any intermediate collection and can appear anywhere in a query.
  * Schema-free: The query language does not assume the existence of a fixed schema for any data it processes.
  * Correlated FROM terms: A right-side FROM term expression can refer to variables defined by FROM terms on its left.
  * Powerful GROUP BY: In addition to a set of aggregate functions as in standard SQL, the groups created by the `GROUP BY` clause are directly usable in nested queries and/or to obtain nested results.
  * Generalized SELECT clause: A SELECT clause can return any type of collection, while in SQL-92, a `SELECT` clause has to return a (homogeneous) collection of records.

> MC: Made it to the end of querying, finally!

# <a id="DDL_and_DML_statements">4. DDL and DML statements</a>

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

In addition to queries, SQL++ supports a variety of statements for data definition and manipulation purposes as well as controlling the context to be used in evaluating SQL++ expressions. AsterixDB supports record-level ACID transactions that begin and terminate implicitly for each record inserted, deleted, or searched while a given SQL++ statement is being executed.

> TW: AsterixDB?
> MC: Good question here - I eradicated the preceding references except in the Intro, which needs a rewrite, but here it is really still about AsterixDB, I think?  (Since most of these statements will be hidden in the Couchbase case?)

This section details the DDL and DML statements supported in the SQL++ language.

## <a id="Declarations">Declarations

    DatabaseDeclaration ::= "USE" Identifier

The world of data in an AsterixDB cluster is organized into data namespaces called databases. To set the default database for a series of statements, the use statement is provided.

As an example, the following statement sets the default database to be "TinySocial".

#### Example

    USE TinySocial;

When writing a complex SQL++ query, it can sometimes be helpful to define one or more
auxilliary functions that each address a sub-piece of the overall query. The declare function statement supports the creation of such helper functions.

    FunctionDeclaration  ::= "DECLARE" "FUNCTION" Identifier ParameterList "{" Expression "}"
    ParameterList        ::= "(" ( <VARIABLE> ( "," <VARIABLE> )* )? ")"

The following is a very simple example of a temporary SQL++ function definition.

#### Example

    DECLARE FUNCTION add(a, b) {
      a + b
    };

## <a id="Life_cycle_management_statements">Lifecycle management statements

    CreateStatement ::= "CREATE" ( DatabaseSpecification
                                 | TypeSpecification
                                 | TableSpecification
                                 | IndexSpecification
                                 | FunctionSpecification )

    QualifiedName       ::= Identifier ( "." Identifier )?
    DoubleQualifiedName ::= Identifier "." Identifier ( "." Identifier )?

The create statement in SQL++ is used for creating persistent artifacts in the context of database. It can be used to create new databases, datatypes, stored collections, indexes, and user-defined SQL++ functions.

#### Databases

    DatabaseSpecification ::= "DATABASE" Identifier IfNotExists ( "WITH" "FORMAT" StringLiteral )?

The create database statement is used to create new databases. To ease the authoring of reusable SQL++ scripts, its optional IfNotExists clause allows creation to be requested either unconditionally or only if the database does not already exist. If this clause is absent, an error will be returned if the specified database already exists. The `WITH FORMAT` clause is a placeholder for future functionality that can safely be ignored.

The following example creates a database named TinySocial.

#### Example

    CREATE DATABASE TinySocial;

#### Types

    TypeSpecification    ::= "TYPE" FunctionOrTypeName IfNotExists "AS" TypeExpr
    FunctionOrTypeName   ::= QualifiedName
    IfNotExists          ::= ( <IF> <NOT> <EXISTS> )?
    TypeExpr             ::= RecordTypeDef | TypeReference | OrderedListTypeDef | UnorderedListTypeDef
    RecordTypeDef        ::= ( <CLOSED> | <OPEN> )? "{" ( RecordField ( "," RecordField )* )? "}"
    RecordField          ::= Identifier ":" ( TypeExpr ) ( "?" )?
    NestedField          ::= Identifier ( "." Identifier )*
    IndexField           ::= NestedField ( ":" TypeReference )?
    TypeReference        ::= Identifier
    OrderedListTypeDef   ::= "[" ( TypeExpr ) "]"
    UnorderedListTypeDef ::= "{{" ( TypeExpr ) "}}"

> TW: How should we refer to the data model? "Asterix Data Model" seems system specific.

The create type statement is used to create a new named ADM datatype. This type can then be used to create stored collections or utilized when defining one or more other ADM datatypes. Much more information about the Asterix Data Model (ADM) is available in the [data model reference guide](datamodel.html) to ADM. A new type can be a record type, a renaming of another type, an ordered list type, or an unordered list type. A record type can be defined as being either open or closed. Instances of a closed record type are not permitted to contain fields other than those specified in the create type statement. Instances of an open record type may carry additional fields, and open is the default for a new type (if neither option is specified).

The following example creates a new ADM record type called GleambookUser type. Since it is closed, its instances will contain only what is specified in the type definition. The first four fields are traditional typed name/value pairs. The friend-ids field is an unordered list of 32-bit integers. The employment field is an ordered list of instances of another named record type, EmploymentType.

#### Example

    CREATE TYPE GleambookUserType AS CLOSED {
      "id" :         int32,
      "alias" :      string,
      "name" :       string,
      "user-since" : datetime,
      "friend-ids" : {{ int32 }},
      "employment" : [ EmploymentType ]
    }

The next example creates a new ADM record type called FbUserType. Note that the type of the id field is UUID. You need to use this field type if you want to have this field be an autogenerated-PK field. Refer to the Tables section later for more details.

#### Example

    CREATE TYPE FbUserType AS CLOSED {
      "id" :         uuid,
      "alias" :      string,
      "name" :       string
    }

#### Tables

    TableSpecification ::= ( <INTERNAL> )? <TABLE> QualifiedName "(" QualifiedName ")" IfNotExists 
                           PrimaryKey ( <ON> Identifier )? ( <HINTS> Properties )? 
                           ( "USING" "COMPACTION" "POLICY" CompactionPolicy ( Configuration )? )?
                           ( <WITH> <FILTER> <ON> Identifier )?
                           |
                           <EXTERNAL> <TABLE> QualifiedName "(" QualifiedName ")" IfNotExists <USING> AdapterName
                           Configuration ( <HINTS> Properties )?
                           ( <USING> <COMPACTION> <POLICY> CompactionPolicy ( Configuration )? )?
    AdapterName          ::= Identifier
    Configuration        ::= "(" ( KeyValuePair ( "," KeyValuePair )* )? ")"
    KeyValuePair         ::= "(" StringLiteral "=" StringLiteral ")"
    Properties           ::= ( "(" Property ( "," Property )* ")" )?
    Property             ::= Identifier "=" ( StringLiteral | IntegerLiteral )
    FunctionSignature    ::= FunctionOrTypeName "@" IntegerLiteral
    PrimaryKey           ::= <PRIMARY> <KEY> NestedField ( "," NestedField )* ( <AUTOGENERATED> )?
    CompactionPolicy     ::= Identifier

> TW: Again, a lot of AsterixDB in the following paragraph.
> Also, while I'm sure that this was always like this, the separation of `Configuration`
> from `Properties` looks pretty confusing ...

The create table statement is used to create a new table. Tables are named, unordered collections of ADM record instances; they are where data lives persistently and are the targets for queries in AsterixDB. Tables are typed, and AsterixDB will ensure that their contents conform to their type definitions. An Internal table (the default) is a table that is stored in and managed by AsterixDB. It must have a specified unique primary key that can be used to partition data across nodes of an AsterixDB cluster. The primary key is also used in secondary indexes to uniquely identify the indexed primary data records. Random primary key (UUID) values can be auto-generated by declaring the field to be UUID and putting "AUTOGENERATED" after the "PRIMARY KEY" identifier. In this case, values for the auto-generated PK field should not be provided by the user since it will be auto-generated by AsterixDB. Optionally, a filter can be created on a field to further optimize range queries with predicates on the filter's field. (Refer to [Filter-Based LSM Index Acceleration](filters.html) for more information about filters.)

> TW: "The Filter-Based LSM Index Acceleration" seems to be quite system specific ...

An External stored collection is stored outside of AsterixDB (currently tables in HDFS or on the local filesystem(s) of the cluster's nodes are supported). External table support allows SQL++ queries to treat external data as though it were stored in AsterixDB, making it possible to query "legacy" file data (e.g., Hive data) without having to physically import it into AsterixDB. For an external table, an appropriate adapter must be selected to handle the nature of the desired external data. (See the [guide to external data](externaldata.html) for more information on the available adapters.)

When creating a stored collection, it is possible to choose a merge policy that controls which of the underlaying LSM storage components to be merged.  Currently, AsterixDB provides four different merge policies that can be configured per table: no-merge, constant, prefix, and correlated-prefix. The no-merge policy simply never merges disk components. While the constant policy merges disk components when the number of components reaches some constant number k, which can be configured by the user. The prefix policy relies on component sizes and the number of components to decide which components to merge. Specifically, it works by first trying to identify the smallest ordered (oldest to newest) sequence of components such that the sequence does not contain a single component that exceeds some threshold size M and that either the sum of the component's sizes exceeds M or the number of components in the sequence exceeds another threshold C. If such a sequence of components exists, then each of the components in the sequence are merged together to form a single component. Finally, the correlated-prefix is similar to the prefix policy but it delegates the decision of merging the disk components of all the indexes in a table to the primary index. When the policy decides that the primary index needs to be merged (using the same decision criteria as for the prefix policy), then it will issue successive merge requests on behalf of all other indexes associated with the same table. The default policy for AsterixDB is the prefix policy except when there is a filter on a table, where the preferred policy for filters is the correlated-prefix.

The following example creates an internal table for storing FacefookUserType records.
It specifies that their id field is their primary key.

#### Example
    CREATE INTERNAL TABLE GleambookUsers(GleambookUserType) PRIMARY KEY id;

The following example creates an internal table for storing FbUserType records. It specifies that their id field is their primary key. It also specifies that the id field is an auto-generated field, meaning that a randomly generated UUID value will be assigned to each record by the system. (A user should therefore not proivde a value for this field.) Note that the id field should be UUID.

#### Example

    CREATE INTERNAL TABLE FbMsgs(FbUserType) PRIMARY KEY id AUTOGENERATED;

The next example creates an external table for storing LineitemType records. The choice of the `hdfs` adapter means that its data will reside in HDFS. The create statement provides parameters used by the hdfs adapter: the URL and path needed to locate the data in HDFS and a description of the data format.

#### Example

    CREATE EXTERNAL TABLE Lineitem('LineitemType) USING hdfs (
      ("hdfs"="hdfs://HOST:PORT"),
      ("path"="HDFS_PATH"),
      ("input-format"="text-input-format"),
      ("format"="delimited-text"),
      ("delimiter"="|"));

#### Indices

    IndexSpecification ::= <INDEX> Identifier IfNotExists <ON> QualifiedName
                           "(" ( IndexField ) ( "," IndexField )* ")" ( "type" IndexType )?
                           ( <ENFORCED> )?
    IndexType          ::= <BTREE> | <RTREE> | <KEYWORD> | <NGRAM> "(" IntegerLiteral ")"

The create index statement creates a secondary index on one or more fields of a specified table. Supported index types include `BTREE` for totally ordered datatypes, `RTREE` for spatial data, and `KEYWORD` and `NGRAM` for textual (string) data. An index can be created on a nested field (or fields) by providing a valid path expression as an index field identifier.

An index field is not required to be part of the datatype associated with a table if that datatype is declared as open and the field's type is provided along with its type and the `ENFORCED` keyword is specified in the end of index definition. `ENFORCING` an open field will introduce a check that will make sure that the actual type of an indexed field (if the field exists in the record) always matches this specified (open) field type.

The following example creates a btree index called gbAuthorIdx on the author-id field of the GleambookMessages table. This index can be useful for accelerating exact-match queries, range search queries, and joins involving the author-id field.

#### Example

    CREATE INDEX gbAuthorIdx ON GleambookMessages(author-id) TYPE BTREE;

The following example creates an open btree index called gbSendTimeIdx on the open send-time field of the GleambookMessages table having datetime type. This index can be useful for accelerating exact-match queries, range search queries, and joins involving the send-time field.

#### Example

    CREATE INDEX gbSendTimeIdx ON GleambookMessages(send-time:datetime) TYPE BTREE ENFORCED;

The following example creates a btree index called twUserScrNameIdx on the screen-name field, which is a nested field of the user field in the ChirpMessages table. This index can be useful for accelerating exact-match queries, range search queries, and joins involving the screen-name field.

#### Example

    CREATE INDEX twUserScrNameIdx ON ChirpMessages(user.screen-name) TYPE BTREE;

The following example creates an rtree index called gbSenderLocIdx on the sender-location field of the GleambookMessages table. This index can be useful for accelerating queries that use the [`spatial-intersect` function](functions.html#spatial-intersect) in a predicate involving the sender-location field.

#### Example

    CREATE INDEX gbSenderLocIndex ON GleambookMessages("sender-location") TYPE RTREE;

The following example creates a 3-gram index called fbUserIdx on the name field of the GleambookUsers table. This index can be used to accelerate some similarity or substring maching queries on the name field. For details refer to the [document on similarity queries](similarity.html#NGram_Index).

#### Example

    CREATE INDEX fbUserIdx ON GleambookUsers(name) TYPE NGRAM(3);

The following example creates a keyword index called fbMessageIdx on the message field of the GleambookMessages table. This keyword index can be used to optimize queries with token-based similarity predicates on the message field. For details refer to the [document on similarity queries](similarity.html#Keyword_Index).

##### Example

    CREATE INDEX fbMessageIdx ON GleambookMessages(message) TYPE KEYWORD;

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

    DROP TABLE GleambookUsers IF EXISTS;

    DROP INDEX gbSenderLocIndex;

    DROP TYPE GleambookUserType;

    DROP DATABASE TinySocial;

    DROP FUNCTION add;

### Import/Export Statements

    LoadStatement  ::= <LOAD> <TABLE> QualifiedName <USING> AdapterName Configuration ( <PRE-SORTED> )?

The load statement is used to initially populate a table via bulk loading of data from an external file. An appropriate adapter must be selected to handle the nature of the desired external data. The load statement accepts the same adapters and the same parameters as external tables. (See the [guide to external data](externaldata.html) for more information on the available adapters.) If a table has an auto-generated primary key field, a file to be imported should not include that field in it.

The following example shows how to bulk load the GleambookUsers table from an external file containing data that has been prepared in ADM format.

##### Example

    LOAD TABLE GleambookUsers USING localfs
    (("path"="localhost:///Users/zuck/AsterixDB/load/fbu.adm"),("format"="adm"));

## <a id="Modification_statements">Modification statements

### <a id="Inserts">Inserts

    InsertStatement ::= <INSERT> <INTO> <TABLE> QualifiedName Query

> TW: AsterixDB-specifc transactions semantics ...
> Also, do we also support `UPSERT`?

The SQL++ insert statement is used to insert data into a table. The data to be inserted comes from a SQL++ query expression. The expression can be as simple as a constant expression, or in general it can be any legal SQL++ query. Inserts in AsterixDB are processed transactionally, with the scope of each insert transaction being the insertion of a single object plus its affiliated secondary index entries (if any). If the query part of an insert returns a single object, then the insert statement itself will be a single, atomic transaction. If the query part returns multiple objects, then each object inserted will be handled independently as a tranaction. If a table has an auto-generated primary key field, an insert statement should not include a value for that field in it. (The system will automatically extend the provided record with this additional field and a corresponding value.)

The following example illustrates a query-based insertion.

#### Example

    INSERT INTO TABLE UsersCopy (FROM GleambookUsers user SELECT VALUE user)

### <a id="Deletes">Deletes

    DeleteStatement ::= <DELETE> <FROM> <TABLE> QualifiedName ( (<AS>)? Variable )? ( <WHERE> Expression )?

The SQL++ delete statement is used to delete data from a target table. The data to be deleted is identified by a boolean expression involving the variable bound to the target table in the delete statement. Deletes in AsterixDB are processed transactionally, with the scope of each delete transaction being the deletion of a single object plus its affiliated secondary index entries (if any). If the boolean expression for a delete identifies a single object, then the delete statement itself will be a single, atomic transaction. If the expression identifies multiple objects, then each object deleted will be handled independently as a transaction.

The following example illustrates a single-object deletion.

#### Example

    DELETE FROM TABLE GleambookUsers user WHERE user.id = 8;

> TW: I think that we now also support this without the `user` variable, right?
> If so, that'd be a good example to have.

We close this guide to SQL++ with one final example of a query expression.

#### Example

    FROM [ "great", "brilliant", "awesome" ] AS praise
    SELECT VALUE `string-concat`(["AsterixDB is ", praise]);
    
