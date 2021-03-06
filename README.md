# IMu API for Perl

At its core, IMu provides a set of Application Programming Interfaces (APIs).

# Contents

* [Using The IMu API](#1-using-the-imu-api)
    * [Test Program](#1-1-test-program)
    * [Exceptions](#1-2-exceptions)
* [Connecting to an IMu server](#2-connecting-to-an-imu-server)
    * [Handlers](#2-1-handlers)
* [Accessing an EMu Module](#3-accessing-an-emu-module)
    * [Searching a Module](#3-1-searching-a-module)
        * [The findKey Method](#3-1-1-the-findkey-method)
        * [The findKeys Method](#3-1-2-the-findkeys-method)
        * [The findTerms Method](#3-1-3-the-findterms-method)
        * [The findWhere Method](#3-1-4-the-findwhere-method)
        * [Number of Matches](#3-1-5-number-of-matches)
    * [Sorting](#3-2-sorting)
        * [The sort Method](#3-2-1-the-sort-method)
    * [Getting Information from Matching Records](#3-3-getting-information-from-matching-records)
        * [The fetch Method](#3-3-1-the-fetch-method)
        * [Specifying Columns](#3-3-2-specifying-columns)
        * [Example](#3-3-3-example)
    * [Multimedia](#3-4-multimedia)
        * [Multimedia Attachments](#3-4-1-multimedia-attachments)
        * [Multimedia Files](#3-4-2-multimedia-files)
        * [Filters](#3-4-3-filters)
        * [Modifiers](#3-4-4-modifiers)
* [Maintaining State](#4-maintaining-state)
    * [Example](#4-1-example)
* [Logging in to an IMu server](#5-logging-in-to-an-imu-server)
    * [The login method](#5-1-the-login-method)
    * [The logout method](#5-2-the-logout-method)
* [Updating an EMu Module](#6-updating-an-emu-module)
    * [The insert Method](#6-1-the-insert-method)
    * [The update Method](#6-2-the-update-method)
    * [The remove Method](#6-3-the-remove-method)
* [Exceptions](#7-exceptions)

<h1 id="1-using-the-imu-api">Using The IMu API</h1>

The IMu Perl API source code bundle for version 2.0 (or higher) is required to develop an IMu-based application. This bundle contains all the modules that make up the IMu Perl API. IMu API bundles are available from the IMu [releases](https://github.com/axiell/imu-api-perl/releases) page.

To build Perl applications the source code bundle must be extracted on the server machine and the path to the source code specified in the **PERL5LIB** environment variable or as an argument to the `lib` module. For example, if the IMu API source code is installed in the directory `/usr/local/lib/imu` the following lines would be added to the Perl code:

```
use lib '/usr/local/lib/imu';
use IMu;
```

<h2 id="1-1-test-program">Test Program</h2>

Compiling and running this very simple IMu program is a good test of whether the development environment has been set up properly for using IMu:

```
#!/usr/bin/perl

use strict;
use warnings;
use lib '/usr/local/lib/imu';
use IMu;

printf("IMu version %s\n", $IMu::VERSION);
exit(0);
```

The `IMu.pm` file defines a class called `IMu`. This class includes the member `VERSION` which contains the version of this IMu release.

<h2 id="1-2-exceptions">Exceptions</h2>

Many of the methods in the IMu library objects throw an exception (`die` in Perl terminology) when an error occurs. For this reason, code that uses IMu library objects should be surrounded with an `eval` block.

The following code is a basic template for writing Java programs that use the IMu library:

```
# ⋯
eval
{
    # Create and use IMu objects
    # ⋯
};
if ($@)
{
    # Handle or report error
    # ⋯
}
```

Most IMu exceptions throw an `IMu::Exception` object. In many cases your code can simply handle the `$@` variable (as in this template). If more information is required about the exact `IMu::Exception` thrown, see [Exceptions](#7-exceptions).

> **NOTE:**
>
> Many of the examples that follow assume that code fragments have been surrounded with code structured in this way.

<h1 id="2-connecting-to-an-imu-server">Connecting to an IMu Server</h1>

Most IMu based programs begin by creating a connection to an IMu server. Connections to a server are created and managed using IMu’s `IMu::Session` class. Before connecting, both the name of the host and the port number to connect on must be specified. This can be done in one of three ways.

1. 
    The simplest way to create a connection to an IMu server is to pass the host name and port number to the `IMu::Session` constructor and then call the `connect` method. For example:

    ```
    use IMu::Session;
    # ⋯
    my $session = IMu::Session->new('server.com', 12345);
    $session->connect();
    ```

1. 
    Alternatively, pass no values to the constructor and then set the `host` and `port` properties (using the `setHost` and `setPort` methods) before calling `connect`:

    ```
    use IMu::Session;
    # ⋯
    my $session = IMu::Session->new();
    $session->setHost('server.com');
    $session->setPort(12345);
    $session->connect();
    ```

1. 
    If either the host or port is not set, the `IMu::Session` class default value will be used. These defaults can be overridden by setting the (static) class properties `defaultHost` and `defaultPort`:

    ```
    use IMu::Session;
    # ⋯
    IMu::Session::setDefaultHost('server.com');
    IMu::Session::setDefaultPort(12345);
    my $session = IMu::Session->new();
    $session->connect();
    ```

    This technique is useful when planning to create several connections to the same server or when wanting to get a [Handler](#2-1-handlers) object to create the connection automatically.

<h2 id="2-1-handlers">Handlers</h2>

Once a connection to an IMu server has been established, it is possible to create handler objects to submit requests to the server and receive responses.

> **NOTE:**
>
> When a handler object is created, a corresponding object is created by the IMu server to service the handler’s requests.

All handlers are subclasses of IMu’s `IMu::Handler` class.

> **NOTE:**
>
> You do not typically create a `IMu::Handler` object directly but instead use a subclass.

In this document we examine the most frequently used handler, `IMu::Module`, which allows you to find and retrieve records from a single EMu module.

<h1 id="3-accessing-an-emu-module">Accessing an EMu Module</h1>

The IMu API provides facilities to search, sort and retrieve information from records in any EMu module. This section contains the reference material for these facilities.

<h2 id="3-1-searching-a-module">Searching a Module</h2>

A program accesses an EMu module (or table, the terms are used interchangeably) using the `IMu::Module` class. The name of the table to be accessed is passed to the `IMu::Module` constructor. For example:

```
use IMu::Module;
# ⋯
my $parties = IMu::Module->new('eparties', $session);
```

This code assumes that a `IMu::Session` object called *session* has already been created. If a `IMu::Session` object is not passed to the `IMu::Module` constructor, a session will be created automatically using the `defaultHost` and `defaultPort` class properties. See [Connecting to an IMu server](#2-connecting-to-an-imu-server) for details.

Once a `IMu::Module` object has been created, it can be used to search the specified module and retrieve records.

Any one of the following methods can be used to search for records within a module:

* [findKey](#3-1-1-the-findkey-method)
* [findKeys](#3-1-2-the-findkeys-method)
* [findTerms](#3-1-3-the-findterms-method)
* [findWhere](#3-1-4-the-findwhere-method)

<h3 id="3-1-1-the-findkey-method">The FindKey Method</h3>

The `findKey` method searches for a single record by its key.

For example, the following code searches for a record with a key of 42 in the Parties module:

```
use IMu::Module;
# ⋯
my $parties = IMu::Module->new('eparties', $session);
my $hits = $parties->findKey(42);
```

The method returns the number of matches found, which is either 1 if the record exists or 0 if it does not.

<h3 id="3-1-2-the-findkeys-method">The FindKeys Method</h3>

The `findKeys` method searches for a set of key values. The keys are passed as an array reference:

```
my $parties = IMu::Module->new('eparties', $session);
my $keys = [52, 42, 17];
my $hits = $parties->findKeys($keys);
```

The method returns the number of records found.

<h3 id="3-1-3-the-findterms-method">The FindTerms Method</h3>

The `findTerms` method is the most flexible and powerful way to search for records within a module. It can be used to run simple single term queries or complex multi-term searches.

The terms are specified using a `IMu::Terms` object. Once a `IMu::Terms` object has been created, add specific terms to it (using the `add` method) and then pass the `IMu::Terms` object to the `findTerms` method. For example, to specify a Parties search for records which contain a first name of “John” and a last name of “Smith”:

```
use IMu::Terms;

my $search = IMu::Terms->new();
$search->add('NamFirst', 'John');
$search->add('NamLast', 'Smith');

my $hits = $parties->findTerms($search);
```

There are several points to note:

1. 
    The first argument passed to the `add` method element contains the name of the column or an alias in the module to be searched.
1. 
    An alias associates a supplied value with one or more actual columns. Aliases are created using the `addSearchAlias` or `addSearchAliases` methods.
1. 
    The second argument contains the value to search for.
1. 
    Optionally, a comparison operator can be supplied as a third argument (see below examples). The operator specifies how the value supplied as the second argument should be matched.

    Operators are the same as those used in TexQL (see KE’s [TexQL documentation](https://emu.kesoftware.com/downloads/Texpress/texql.pdf) for details). If not supplied, the operator defaults to “matches”.

    This is not a real TexQL operator, but is translated by the search engine as the most “natural” operator for the type of column being searched. For example, for *text* columns “matches” is translated as the `contains` TexQL operator and for *integer* columns it is translated as the `=` TexQL operator.

> **NOTE:**
>
>  Unless it is really necessary to specify an operator, consider using the `matches` operator, or better still supplying no operator at all as this allows the server to determine the best type of search.

**Examples**

1. 
    To search for the name “Smith” in the last name field of the Parties module, the following term can be used:

    ```
    my $search = IMu::Terms->new();
    $search->add('NamLast', 'Smith');
    ```

1. 
    Specifying search terms for other types of columns is straightforward. For example, to search for records inserted on April 4, 2011:

    ```
    my $search = IMu::Terms->new();
    $search->add('AdmDateInserted', 'Apr 4 2011');
    ```

1. 
    To search for records inserted before April 4, 2011, it is necessary to add an operator:

    ```
    my $search = IMu::Terms->new();
    $search->add('AdmDateInserted', 'Apr 4 2011', '<');
    ```

1. 
    By default, the relationship between the terms is a Boolean `AND`. This means that to find records which match both a first name containing “John” and a last name containing “Smith” the `IMu::Terms` object can be created as follows:

    ```
    my $search = IMu::Terms->new();
    $search->add('NamFirst', 'John');
    $search->add('NamLast', 'Smith');
    ```

1. 
    A `IMu::Terms` object where the relationship between the terms is a Boolean `OR` can be created by passing the value "OR" to the `IMu::Terms` constructor:

    ```
    my $search = IMu::Terms->new('OR');
    $search->add('NamFirst', 'John');
    $search->add('NamLast', 'Smith');
    ```

    This specifies a search for records where either the first name contains “John” or the last name contains “Smith”.

1. 
    Combinations of `AND` and `OR` search terms can be created. The `addAnd` method adds a new set of `AND` terms to the original `IMu::Terms` object. Similarly the `addOr` method adds a new set of `OR` terms. For example, to restrict the search for a first name of “John” and a last name of “Smith” to matching records inserted before April 4, 2011 or on May 1, 2011, specify:

    ```
    my $search = IMu::Terms->new();
    $search->add('NamFirst', 'John');
    $search->add('NamLast', 'Smith');
    my $dates = $search->addOr();
    $dates->add('AdmDateInserted', 'Apr 4 2011', '<');
    $dates->add('AdmDateInserted', 'May 1 2011');
    ```

1. 
    To run a search, pass the `IMu::Terms` object to the `findTerms` method:

    ```
    my $parties = IMu::Module->new('eparties', $session);
    my $search = IMu::Terms->new();
    $search->add('NamLast', 'Smith');
    my $hits = $parties->findTerms($search);
    ```

    As with other find methods, the return value contains the estimated number of matches.

1. 
    To use a search alias, call the `addSearchAlias` method to associate the alias with one or more real column names before calling `findTerms`. Suppose we want to allow a user to search the Catalogue module for keywords. Our definition of a keywords search is to search the *SummaryData*, *CatSubjects_tab* and *NotNotes* columns. We could do this by building an `OR` search:

    ```
    my $keyword = '⋯';
    # ⋯
    my $search = IMu::Terms->new('OR');
    $search->add('SummaryData', $keyword);
    $search->add('CatSubjects_tab', $keyword);
    $search->add('NotNotes', $keyword);
    ```

    Another way of doing this is to register the association between the name *keywords* and the three columns we are interested in and then pass the name *keywords* as the column to be searched:

    ```
    my $keyword = '⋯';
    # ⋯
    my $catalogue = IMu::Module->new('ecatalogue', $session);
    my $columns =
    [
        'SummaryData',
        'CatSubjects_tab',
        'NotNotes'
    ];
    $catalogue->addSearchAlias('keywords', $columns);
    # ⋯
    my $search = IMu::Terms->new();
    $search->add('keywords', $keyword);
    $catalogue->findTerms($search);
    ```

    An alternative to passing the columns as an array of strings is to pass a single string, with the column names separated by semi-colons:

    ```
    my $keyword = '⋯';
    # ⋯
    my $catalogue = IMu::Module->new('ecatalogue', $session);
    my $columns = 'SummaryData;CatSubjects_tab;NotNotes';
    $catalogue->addSearchAlias('keywords', $columns);
    # ⋯
    my $search = IMu::Terms->new();
    $search->add('keywords', $keyword);
    $catalogue->findTerms($search);
    ```

    The advantage of using a search alias is that once the alias is registered a simple name can be used to specify a more complex `OR` search.

1. 
    To add more than one alias at a time, use the IMu `Map` class to build an associative array of names and columns and call the `addSearchAliases` method:

    ```
    my $aliases =
    {
        'keywords' => 'SummaryData;CatSubjects_tab;NotNotes',
        'title' => 'SummaryData;TitMainTitle'
    };
    $catalogue->addSearchAliases($aliases);
    ```

<h3 id="3-1-4-the-findwhere-method">The FindWhere Method</h3>

With the `findWhere` method it is possible to submit a complete TexQL *where* clause:

```
my $parties = IMu::Module->new('eparties', $session);
my $where = "NamLast contains 'Smith'";
my $hits = $parties->findWhere($where);
```

Although this method provides complete control over exactly how a search is run, it is generally better to use `findTerms` to submit a search rather than building a *where* clause. There are (at least) two reasons to prefer `findTerms` over `findWhere`:

1. 
    Building the *where* clause requires the code to have detailed knowledge of the data type and structure of each column. The `findTerms` method leaves this task to the server.

    For example, specifying the term to search for a particular value in a nested table is straightforward. To find Parties records where the Roles nested table contains Artist, `findTerms` simply requires:
    
    ```
    $search->add('NamRoles_tab', 'Artist');
    ```

    On the other hand, the equivalent TexQL clause is:

    ```
    exists(NamRoles_tab where NamRoles contains 'Artist');
    ```

    The TexQL for double nested tables is even more complex.

1. 
    More importantly, findTerms is more secure.

    With `findTerms` a set of terms is submitted to the server which then builds the TexQL *where* clause. This makes it much easier for the server to check for terms which may contain SQL-injection style attacks and to avoid them.
    
    If your code builds a *where* clause from user-entered data so it can be run using `findWhere`, it is much more difficult, if not impossible, for the server to check and avoid SQL-injection. The responsibility for checking for SQL-injection becomes yours.

<h3 id="3-1-5-number-of-matches">Number of Matches</h3>

All of the *find* methods return the number of matches found by the search. For `findKey` and `findKeys` this number is always the exact number of matches found. The number returned by `findTerms` and `findWhere` is best thought of as an estimate.

This estimate is almost always correct but because of the nature of the indexing used by the server’s data engine (Texpress) the number can sometimes be an over-estimate of the real number of matches. This is similar to the estimated number of hits returned by a Google search.

<h2 id="3-2-sorting">Sorting</h2>

<h3 id="3-2-1-the-sort-method">The sort Method</h3>

The `IMu::Module` class `sort` method is used to order a set of matching records once the search of a module has been run.

<h4 id="3-2-1-1-arguments">Arguments</h4>

This `sort` method takes two arguments:

* **columns**

    The *columns* argument is used to specify the columns by which to sort the result set. The value of the argument can be either a string, or a reference to an array of strings. Each string can be a simple column name or a set of column names, separated by semi-colons or commas.

    Each column name can be preceded by a `+` (plus) or `-` (minus or dash). A leading `+` indicates that the records should be sorted in ascending order. A leading `-` indicates that the records should be sorted in descending order.

    > **NOTE:**
    >
    > If a sort order (“+” or “-”) is not given, the sort order defaults to ascending.

* **flags**

    The *flags* argument is used to pass one or more flags to control the way the sort is carried out. As with the *columns* argument, the *flags* argument can be a stringor a reference to an array of strings. Each string can be a single flag or a set of flags separated by semi-colons or commas.

    The following flags control the type of comparisons used when sorting:

    * **word-based**

        Sort disregards white spaces (more than the one space between words). As of Texpress 9.0, `word-based` sorting no longer disregards punctuation. For example:
    
        > Traveler's&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Inn

        will be sorted as

        > Traveler's Inn

    * **full-text**

        Sort includes all punctuation and white spaces. For example:

        > Traveler's&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Inn

        will be sorted as

        > Traveler's&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Inn

        and will therefore differ from

        > Traveler's Inn

    * **compress-spaces**
    
        Sort includes punctuation but disregards all white space (with the exception of a single space between words). For example:

        > Traveler's&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Inn

        will be sorted as

        > Traveler's Inn

        > **NOTE:**
        >
        > If none of these flags are included, the comparison defaults to word-based.

    The following flags modify the sorting behaviour:

    * **case-sensitive**

        Sort is sensitive to upper and lower case. For example:

        > Melbourne gallery

        will be sorted separately to

        > Melbourne Gallery

    * **order-insensitive**

        Values in a multi-value field will be sorted alphabetically regardless of the order in which they display. For example, a record which has the following values in the *NameRoles_tab* column in this order:

        1. Collection Manager
        1. Curator
        1. Internet Administrator

        and another record which has the values in this order:

        1. Internet Administrator
        1. Collection Manager
        1. Curator

        will be sorted the same.

    * **null-low**

        Records with empty columns will be placed at the start of the result set rather than at the end.

    * **extended-sort**

        Values that include diacritics will be sorted separately to those that do not. For example:

        > entrée

        will be sorted separately to

        > entree

    The following flags can be used when generating a summary of the sorted records:

    * **report**

        A summary of the sort is generated. The summary report is a hierarchically structured object that summarises the number of unique values contained in the sort columns. See [Return Value](#3-2-2-return-value) and [Example](#3-2-3-example) for a description and illustration of the returned structure.

    * **table-as-text**

        All data from multi-valued columns will be treated as a single value (joined by line break characters) in the summary results array. For example, for a record which has the following values in the NamRoles_tab column:

        > Collection Manager, Curator, Internet Administrator

        the summary will include statistics for a single value:

        > Collection Manager
        > Curator
        > Internet Administrator

        Thus the number of values in the summary results display will match the number of records.

        If this option is not included, each value in a multi-valued column will be treated as a distinct value in the summary. Thus there may be many more values in the summary results than there are records.

For example:

1. 
    Sort parties by first name (ascending):

    ```
    my $parties = IMu::Module->new('eparties', $session);
    my $search = IMu::Terms->new();
    $search->add('NamLast', 'Smith');
    my $hits = $parties->findTerms($search);

    $parties->sort('NamFirst');
    ```

1. 
    Sort parties by title (ascending) and then first name (descending):

    ```
    # ⋯
    my $sort =
    [
        'NamTitle',
        '-NamFirst'
    ];
    $parties->sort($sort);
    ```

1. 
    Run a case-sensitive sort of parties by title (ascending) and then first name (descending):

    ```
    # ⋯
    my $sort =
    [
        'NamTitle',
        '-NamFirst'
    ];
    my $flags =
    [
        'case-sensitive'
    ];
    $parties->sort($sort, $flags);
    ```

<h4 id="3-2-1-2-return-value">Return Value</h4>

The `sort` method returns `null` unless the *report* flag is used.

If the *report* flag is used, the `sort` method returns a simple array reference representing a list of distinct terms associated with the primary column in the sorted result set.

Each element in the array is a hash. The hash contains three elements which describe the term:

*
    **value**

    A string that is the distinct value itself.

*
    **count**

    An integer specifying the number of records in the result set which have the *value*.

*
    **list**

    A nested array reference that holds the values for secondary sorts within the primary sort.

This is illustrated in the following example.

<h4 id="3-2-1-3-example">Example</h4>

This example shows a three-level sort by title, last name (descending) and first name on a set of Parties records:

```
my $parties = IMu::Module->new('eparties', $session);

my $terms = IMu::Terms->new('or');
$terms->add('NamLast', 'Smith');
$terms->add('NamLast', 'Wood');

my $hits = $parties->findTerms($terms);

my $sort =
[
    'NamTitle',
    '-NamLast',
    'NamFirst'
];
my $flags =
[
    'full-text',
    'report'
];
my $report = $parties->sort($sort, $flags);
showSummary($report, 0);
exit(0);

sub showSummary
{
    my $report = shift;
    my $indent = shift;

    if (! defined($report))
    {
        return;
    }
    my $prefix = '  ' x $indent;

    foreach my $term (@$report)
    {
        my $value = $term->{'value'};
        my $count = $term->{'count'};
        printf("%s%s (%d)\n", $prefix, $value, $count);

        my $list = $term->{'list'};
        showSummary($list, $indent + 1);
    }
}
```

This displays the distinct terms (and their counts) for the primary sort key (title). Nested under each primary key is the set of distinct terms and counts for the secondary key (last name) and nested under each secondary key is the set of distinct terms and counts for the tertiary key (first name):

```
Mr (2)
  Wood (1)
    Gerard (1)
  SMITH (1)
    Ian (1)
Ms (1)
  ECCLES-SMITH (1)
    Kate (1)
Sir (1)
  Wood (1)
    Henry (1)
 (3)
  Wood (1)
    Grant (1)
  Smith (2)
    Sophia (1)
    William (1)
```

If another sort key was specified its terms would be nested under the tertiary key and so on.

> **NOTE:**
>
> In the example above some of the records do not have a value for the primary sort key (title). By default these values are sorted after any other values. They can be sorted before other values using the *null-low* flag.

<h2 id="3-3-getting-information-from-matching-records">Getting Information from Matching Records</h2>

<h3 id="3-3-1-the-fetch-method">The fetch Method</h3>

The `IMu::Module` class `fetch` method is used to get information from the matching records once the search of a module has been run. The server maintains the set of matching records in a list and the `fetch` method can be used to retrieve any information from any contiguous block of records in the list.

#### Arguments

The `fetch` method has four arguments:

* 
    **flag**

* 
    **offset**

    Together the *flag* and *offset* arguments define the starting position of the block of records to be fetched. The *flag* argument is a string and must be one of:

    * start
    * current
    * end

    The “start” and “end” flags refer to the first record and the last record in the matching set. The “current” flag refers to the position of the last record fetched by the previous call to the `fetch` method. If the `fetch` method has not been called, “current” refers to the first record in the matching set.

    The *offset* argument is an `int`. It adjusts the starting position relative to the value of the *flag* argument. A positive value for *offset* specifies a start after the position specified by *flag* and a negative value specifies a start before the position specified by *flag*.

    For example, calling `fetch` with a *flag* of “start” and *offset* of 3 will return records starting from the fourth record in the matching set. Specifying a *flag* of “end” and an *offset* of -8 will return records starting from the ninth last record in the matching set.

    To retrieve the next record after the last returned by the previous `fetch`, you would pass a *flag* of “current” and an *offset* of 1.

* 
    **count**

    The *count* argument specifies the maximum number of records to be retrieved.

    Passing a count value of 0 is valid. This causes `fetch` to change the current record without actually retrieving any data.

    Using a negative value for *count* is also valid. This causes `fetch` to return all the records in the matching set from the starting position (specified by *flag* and *offset*).

* 
    **columns**

    The *columns* argument is used to specify which columns should be included in the returned records. The argument can be either a string or a reference to an array of strings. In its simplest form each string contains a single column name, or several column names separated by semi-colons or commas.

    The value of the *columns* argument can be more than simple column names. See the section on [Specifying Columns](#3-3-2-specifying-columns) for details.

There are a number of variations of the `fetch` method. See the [reference documentations](TODO-link-to-reference) for details of each one.

For example:

1. 
    Retrieve the first record from the start of a set of matching records:

    ```
    my $parties = IMu::Module->new('eparties', $session);
    my $columns = 'NamFirst,NamLast';
    my $result = $parties->fetch('start', 0, 1, $columns);
    ```

    The *columns* argument can also be specified as an array reference:

    ```
    my $parties = IMu::Module->new('eparties', $session);
    my $columns =
    [
        'NamFirst',
        'NamLast'
    ];
    my $result = $parties->fetch('start', 0, 1, $columns);
    ```

1. 
    Return all of the results in a matching set:

    ```
    my $parties = IMu::Module->new('eparties', $session);
    my $columns =
    [
        'NamFirst',
        'NamLast'
    ];
    my $result = $parties->fetch('start', 0, -1, $columns);
    ```

1. 
    Change the current record to the next record in the set of matching records without retrieving any data:

    ```
    $parties->fetch('current', 1, 0);
    ```

1. 
    Retrieve the last record from the end of a set of matching records:

    ```
    my $parties = IMu::Module->new('eparties', $session);
    my $columns =
    [
        'NamFirst',
        'NamLast'
    ];
    my $result = $parties->fetch('end', 0, 1, $columns);
    ```

#### Return Value

The `fetch` method returns records requested in an [ModuleFetchResult](TODO-link-to-reference) object. It contains three members:

* 
    **count**

    An number of records returned by the `fetch` request.

* 
    **hits**

    The estimated number of matches in the result set. This number is returned for each `fetch` because the estimate can decrease as records in the result set are processed by the `fetch` method.

* 
    **rows**

    A reference to an array containing the set of records requested. Each element of the rows array is itself a reference to a hash. Each hash contains entries for each column requested.

For example, retrieve the *count* & *hits* properties and iterate over all of the returned records using the *rows* property:

```
my $columns = 
[
    'NamFirst',
    'NamLast',
];
my $result = $parties->fetch('start', 0, 2, $columns);
my $count = $result->{'count'};
my $hits = $result->{'hits'};

print('Count: ', $count, "\n");
print('Hits: ', $hits, "\n");
print('Rows:', "\n");
foreach my $row (@{$result->{'rows'}})
{
    my $rowNum = $row->{'rownum'};
    my $irn = $row->{'irn'};
    my $firstName = $row->{'NamFirst'};
    my $lastName = $row->{'NamLast'};
    printf("  %d. %s, %s (%d)\n", $rowNum, $lastName, $firstName, $irn);
}
```

This will produce output similar to the following:

```
Count: 2
Hits: 4
Rows:
  1. ECCLES-SMITH, Kate (100573)
  2. SMITH, Ian (100301)
```

<h3 id="3-3-2-specifying-columns">Specifying Columns</h3>

This section specifies the values that can be included or used as the *columns* arguments to the `IMu::Module` class `fetch` method.

<h4 id="3-3-2-1-atomic-columns">Atomic Columns</h4>

These are simple column names of the type already mentioned, for example:

```
NamFirst
```

The values of atomic columns are returned as `Strings`:

```
my $columns =
[
    'NamFirst'
];
my $result = $parties->fetch('start', 0, 1, $columns);
foreach my $row (@{$result->{'rows'}})
{
    my $first = $row->{'NamFirst'};
    # ⋯
}
```

<h4 id="3-3-2-2-nesting-tables">Nested Tabes</h4>

Nested tables are columns that contain a list of values. They are specified similarly to atomic columns:

```
NamRoles_tab
```

The values of nested tables are returned as an array reference. Each array member is a string that corresponds to a row from the nested table:

```
my $columns =
[
    'NamRoles_tab'
];
my $result = $parties->fetch('start', 0, 1, $columns);
foreach my $row (@{$result->{'rows'}})
{
    my $roles = $row->{'NamRoles_tab'};
    foreach my $role (@$roles)
    {
        # ⋯
    }
}
```

<h4 id="3-3-2-3-columns-from-attached-records">Columns From Attached Records</h4>

An attachment is a link between a record in a module and a record in the same or another module. The columns from an attached record can be specified by first specifying the attachment column and then the column to retrieve from the attached record:

```
SynSynonymyRef_tab.SummaryData
```

Multiple columns can be specified from the attached record:

```
SynSynonymyRef_tab.(NamFirst,NamLast)
```

The return values of columns from attached records depends on the type of the attachment column. If the attachment column is atomic then the column values are returned as a hash reference. If the attachment column is a nested table the values are returned as an array reference. Each array member is a hash reference containing the requested column values for each attached record:

```
my $columns =
[
    'SynSynonymyRef_tab.(NamFirst,NamLast)'
];
my $result = $parties->fetch('start', 0, 1, $columns);
foreach my $row (@{$result->{'rows'}})
{
    my $synonymies = $row->{'SynSynonymyRef_tab'};
    foreach my $synonymy (@$synonymies)
    {
        my $first = $synonymy->{'NamFirst'};
        my $last = $synonymy->{'NamLast'};
        # ⋯
    }
}
```

<h4 id="3-3-2-4-columns-from-reverse-attachments">Columns from Reverse Attachments</h4>

A reverse attachment allows you to specify columns from other records in the same module or other modules that have the current record attached to a specified column.

For example:

1. 
    Retrieve the *TitMainTitle* (Main Title) column for all Catalogue records that have the current Parties record attached to their *CreCreatorRef_tab* (Creator) column:

    ```
    <ecatalogue:CreCreatorRef_tab>.TitMainTitle
    ```

1. 
    Retrieve the *NarTitle* (Title) column for all Narratives records that have the current Narrative record attached to their *HieChildNarrativesRef_tab* (Child Narratives) column:

    ```
    <enarratives:HieChildNarrativesRef_tab>.NarTitle
    ```

Multiple columns can be specified from the reverse attachment record:

```
<ecatalogue:CreCreatorRef_tab>.(TitMainTitle,TitObjectCategory)
```

Reverse attachments are returned as an array reference. Each array member is a hash reference containing the requested column values from each record from the specified module (The Catalogue module in the example below) that has the current record attached to the specified column (The *CreCreatorRef_tab* column in the example below):

```
my $columns =
[
    '<ecatalogue:CreCreatorRef_tab>.(TitMainTitle,TitObjectCategory)'
];
my $result = $parties->fetch('start', 0, 1, $columns);
foreach my $row (@{$result->{'rows'}})
{
    my $objects = $row->{'ecatalogue:CreCreatorRef_tab'};
    foreach my $object (@$objects)
    {
        my $title = $object->{'TitMainTitle'};
        my $category = $object->{'TitObjectCategory'};
        # ⋯
    }
}
```

<h4 id="3-3-2-5-grouped-nested-tables">Grouped Nested Tables</h4>

A set of nested table columns can be grouped by specifying them between square brackets.

For example, to group the Contributors and their Role from the Narratives module:

```
[NarContributorRef_tab.SummaryData,NarContributorRole_tab]
```

Each corresponding rows of the supplied nested tables are returned as a single table row in the returned results. By default, the group is given a name of *group1*, *group2* and so on. This group name can be changed by prefixing the grouped columns with an alternative name:

```
contributors=[NarContributorRef_tab.SummaryData,NarContributorRole_tab]
```

The grouped nested tables are returned as an array reference. Each array member is a hash reference containing corresponding rows from the nested tables:

```
my $columns =
[
    '[NarContributorRef_tab.SummaryData,NarContributorRole_tab]'
];
my $result = $narratives->fetch('start', 0, 1, $columns);
foreach my $row (@{$result->{'rows'}})
{
    my $groupedRows = $row->{'group1'};
    foreach my $groupedRow (@$groupedRows)
    {
        # NarContributorRef_tab is an attachment column.
        my $contributor = $groupedRow->{'NarContributorRef_tab'};
        my $summary = $contributor->{'SummaryData'};
        my $role = $groupedRow->{'NarContributorRole_tab'};
        # ⋯
    }
}
```

<h4 id="3-3-2-6-virtual-columns">Virtual Columns</h4>

Virtual columns are columns that do not actually exist in the EMu table being accessed. Instead, the IMu server interprets the request for the column and builds an appropriate response. Certain virtual columns can only be used in certains modules as follows:

**The following virtual columns can be used in any EMu module:**

* 
    insertedTimeStamp

    Returns the insertion date and time of the record using the format `YYY-MM-DDThh:mm:ss`, for example "1999-12-31T23:59:59". This is similar to the [ISO8601](http://en.wikipedia.org/wiki/ISO_8601) date format except the time zone designator is not included.

* 
    modifiedTimeStamp

    Returns the modification date and time of the record using the format `YYYY-MM-DDThh:mm:ss`.

    ```
    my $columns =
    [
        'insertedTimeStamp',
        'modifiedTimeStamp'
    ];
    my $result = $parties->fetch('start', 0, 1, $columns);
    foreach my $row (@{$result->{'rows'}})
    {
        my $inserted = $row->{'insertedTimeStamp'};
        my $modified = $row->{'modifiedTimeStamp'};
        # ⋯
    }
    ```

**The following virtual columns can be used in any EMu module except Multimedia:**

* 
    application

    Returns information about the preferred [application](GLOSSARY.md#application) multimedia attached to a record.

    > **NOTE:**
    >
    > Currently the preferred multimedia is the same as the first entry in the list returned by the *applications* virtual column. However, future versions of EMu may allow other multimedia to be flagged as preferred, in which case the *application* column will return information for the preferred multimedia, rather than the first one.

* 
    applications

    Returns information about all of the application multimedia attached to a record.

* 
    audio

    Returns information about the preferred [audio](GLOSSARY.md#audio) multimedia attached to a record.

* 
    audios

    Returns information about all of the audio multimedia attached to a record.

* 
    image

    Returns information about the preferred [image](GLOSSARY.md#image) multimedia attached to a record.

* 
    images

    Returns information about all of the image multimedia attached to a record.

* 
    multimedia

    Returns information about all of the multimedia attached to a record.

* 
    video

    Returns information about the preferred [video](GLOSSARY.md#video) multimedia attached to a record.

* 
    videos

    Returns information about all of the video multimedia attached to a record.

See [Multimedia](#3-4-multimedia) for more information.

**The following virtual columns can only be used in the Multimedia module:**

* 
    master
    
    Returns information about the [master](GLOSSARY.md#master) multimedia file.

* 
    resolutions

    Returns information about all multimedia [resolutions](GLOSSARY.md#resolution).

* 
    resource

    Returns minimal information about the master multimedia file including an open file handle to a temporary copy of the multimedia file.

* 
    resource

    Returns minimal information about the master multimedia file including an open file handle to a temporary copy of the multimedia file.

* 
    resources

    The same as the 8resource* virtual column except that information and file handles are supplied for all multimedia files.

* 
    supplementary

    Returns information about all [supplementary](GLOSSARY.md#supplementary) multimedia files.

* 
    thumbnail

    Returns information about the multimedia [thumbnail](GLOSSARY.md#thumbnail).

See [Multimedia](#3-4-multimedia) for more information.

**The following virtual column can only be used in the Narratives module:**

* 
    trails

    Returns information about the position of current Narratives record in the narratives hierarchy.

**The following virtual column can only be used in the Collection Descriptions module:**

* extUrlFull_tab

<h4 id="3-3-2-7-fetch-sets">Fetch Sets</h4>

A fetch set allows you to pre-register a group of columns by a single name. That name can then be passed to the `fetch` method to retrieve the specified columns.

Fetch sets are useful if the `fetch` method will be called several times with the same set of columns because:

* The required columns do no have to be specified every time the `fetch` method is called. This is useful when [maintaining state](#4-maintaining-state).

* Every time the `fetch` method is called the IMu server must parse the supplied columns and check them against the EMu schema. For complex column sets, particularly those involving several references or reverse references, this can take time.

The `IMu::Module` class `addFetchSet` method is used to register a set of columns. This method takes two arguments:

* 
    **name**

    The name to use for the column set. The value of this argument can be passed to any call to the `fetch` method and the set of columns specified by the *columns* argument will be returned.

* 
    **columns**

    The set of columns to be associated with the name argument.

The `IMu::Module` class `addFetchSets` method is similar except that multiple sets can be registered at one time.

The results are returned as if you had supplied the columns directly to the `fetch` method.

For example:

1. 
    Add a single fetch set using the `addFetchSet` method:

    ```
    my $columns =
    [
        'NamFirst',
        'NamLast',
        'NamRoles_tab'
    ];
    $parties->addFetchSet('person_details', $columns);
    ```

1. 
    Add multiple fetch sets using the `addFetchSets` method:

    ```
    my $sets =
    {
        'person_details' => ['NamFirst', 'NamLast', 'NamRoles_tab'],
        'organisation_details' => ['NamOrganisation', 'NamOrganisationAcronym']
    };
    $module->addFetchSets($sets);
    ```

1. 
    Retrieve a fetch set using the `fetch` method:

    ```
    my $result = $parties->fetch('start', 0, 1, 'person_details');
    foreach my $row (@{$result->{'rows'}})
    {
        my $first = $row->{'NamFirst'};
        my $last = $row->{'NamLast'};

        my $roles = $row->{'NamRoles_tab'};
        foreach my $role (@$roles)
        {
            # ⋯
        }
    }
    ```

    > **WARNING:**
    >
    >The fetch set name must be the **only** value passed as the `fetch` method *columns* argument. This may be revised in a future version of the IMu API.

<h4 id="3-3-2-8-renaming-columns">Renaming columns</h4>

Columns can be renamed in the returned results by prefixing them with an alternative name:

```
first_name=NamFirst
```

The value of the specified column is now returned using the alternative name:

```
my $columns =
[
    'first_name=NamFirst'
];
my $result = $parties->fetch('start', 0, 1, $columns);
foreach my $row (@{$result->{'rows'}})
{
    my $first = $row->{'first_name'};
    # ⋯
}
```

Alternative names can be supplied to other column specifications as well:

```
roles=NamRoles_tab
```

```
synonyms=SynSynonymyRef_tab.(NamFirst,NamLast)
```

```
objects=<ecatalogue:CreCreatorRef_tab>.(TitMainTitle,TitObjectCategory)
```

Alternative names can also be used for any columns specified between round brackets or square brackets:

```
SynSynonymyRef_tab.(first_name=NamFirst,last_name=NamLast)
```

```
contributors=[contributor=NarContributorRef_tab.SummaryData,role=NarContributorRole_tab]
```

Alternative names can also be supplied in fetch sets:

```
my $columns =
[
    'first_name=NamFirst',
    'last_name=NamLast',
    'roles=NamRoles_tab'
];
$parties->addFetchSet('person_details', $columns);
```

<h3 id="3-3-3-example">Example</h3>

In this example we build a simple Perl <abbr title="Common Gateway Interface">CGI</abbr>-based web page to search the Parties module by last name and display the full set of results.

First build the search page, which is a plain HTML form:

```
<head>
  <title>Party Search</title>
</head>
<body>
  <form action="a-simple-example.jsp">
    <p>Enter a last name to search for (e.g. Smith):</p>
    <input type="text" name="name"/>
    <input type="submit" value="Search"/>
  </form>
</body>
```

Next build the results page, which runs the search and displays the results:

```
#!/usr/bin/env perl

use strict;
use warnings;
use FindBin qw($RealBin);
use lib "$RealBin/../../lib";
use IMu::Module;
use IMu::Session;
use IMu::Terms;
use CGI;
use CGI::Carp qw(fatalsToBrowser);

my $cgi = CGI->new();
my $name = $cgi->param('name');
if (! defined($name) || $name eq '')
{
    print($cgi->header('text/html','400 Bad Request'));
    die("missing 'name' parameter\n");
}
my $terms = IMu::Terms->new();
$terms->add('NamLast', $name);

my $session = IMu::Session->new('imu.mel.kesoftware.com', 40136);
my $parties = IMu::Module->new('eparties', $session);
my $hits = $parties->findTerms($terms);

my $columns =
[
    'NamFirst',
    'NamLast'
];
my $results = $parties->fetch('start', 0, -1, $columns);

print($cgi->header());
print($cgi->start_html(-title => 'IMu Perl API - A Simple Example'));       
print($cgi->p("Number of matches: $hits"));
print($cgi->start_table());

foreach my $row (@{$results->{'rows'}})
{
    my $rowNum = $row->{'rownum'};
    my $firstName = $row->{'NamFirst'};
    my $lastName = $row->{'NamLast'};

    print($cgi->start_Tr()); 
    print($cgi->start_td(), "$rowNum.", $cgi->end_td());
    print($cgi->start_td(), "$lastName, $firstName", $cgi->end_td());
    print($cgi->end_Tr());
}

print($cgi->end_table(), "\n");
print($cgi->end_html());
exit(0);
```

In this example the *name* parameter entered via the HTML search page is submitted to the Perl CGI script. The script searches for parties records that have the entered value as a last name and display the parties first and last names in an HTML table.

For more informaiton about the Perl CGI module & running CGI under apache see:

* http://perldoc.perl.org/CGI.html
* http://perldoc.perl.org/CGI/Carp.html
* http://httpd.apache.org/docs/2.2/howto/cgi.html

<h2 id="3-4-multimedia">Multimedia</h2>

The IMu API provides a number of special mechanisms to handle access to the multimedia stored in the EMu <abbr title="Database management system">DBMS</abbr>. These machanisms fall into three rough categories:

1. 
    Mechanisms to select Multimedia module records that are attached to another module. This is covered in the [Multimedia Attachments](#3-4-1-multimedia-attachments) section.
1. 
    Mechanisms to select multimedia files from a Multimedia module record. This is covered in the [Multimedia Files](#3-4-2-multimedia-files) and [Filters](3-4-3-filters) sections.
1. 
    Mechanisms to apply modifications to multimedia files. This is covered in the [Modifiers](#3-4-4-modifiers) section.

It is important to note that a single record in the EMu DBMS can have multiple Multimedia module records associated with it. Each Multimedia module record can have multiple multimedia files associated with it. The seperate mechanisms for handling multimedia access can be composed so that it is possible to, for example:

* Select a specific Multimedia module record from a group of attached Multimedia module records.
* Select a specific multimedia file from the selected Multimedia record.
* Apply a modification to the selected multimedia file.

<h3 id="3-4-1-multimedia-attachments">Multimedia Attachments</h3>

Information about the multimedia attached to an EMu record from any module (**except** the Multimedia module itself) can be retrieved using the `IMu::Module` class `fetch` method by specifying one of the following [virtual columns](#3-3-2-6-virtual-columns).

The following virtual columns return information about a single multimedia attachment of the current record. The information is returned as a associative array:

* application
* audio
* image
* video

The following virtual columns return information about a set of multimedia attachments of the current record. The information is returned as an array. Each array member is an associative array containing the information for a single multimedia attachment from the set:

* applications
* audios
* images
* multimedia
* videos

All of these virtual columns return the [irn](GLOSSARY.md#irn), [type](GLOSSARY.md#mime-type) and [format](GLOSSARY.md#mime-format) of the Multimedia record attached to the current record. They also act as reference columns to the Multimedia module. This means that other columns from the Multimedia module (including [virtual columns](#3-3-2-6-virtual-columns)) can also be requested from the corresponding Multimedia record, for example:

1. 
    Include the title for all attached multimedia:

    ```
    multimedia.MulTitle
    ```

1. 
    Include the title for all attached images:

    ```
    images.MulTitle
    ```

1. 
    Include details about the master multimedia file for all attached images (using the virtual Multimedia module column *master*):

    ```
    images.master
    ```

1. 
    Include multiple columns for all attached images:

    ```
    images.(master,MulTitle,MulDescription)
    ```

1. 
    Include and rename multiple columns for all attached images:

    ```
    images.(master,title=MulTitle,description=MulDescription)
    ```

<h4 id="3-4-1-1-example">Example</h4>

This example shows the retrieval of the base information and the title for all multimedia images attached to a parties record:

```
my $columns = 
[
    'images.MulTitle'
];
my $result = $parties->fetch('start', 0, 1, $columns);
foreach my $row (@{$result->{'rows'}})
{
    foreach my $image (@{$row->{'images'}})
    {
        my $irn = $image->{'irn'};
        my $type = $image->{'type'};
        my $format = $image->{'format'};
        my $title = $image->{'MulTitle'};

        printf("irn %d: %s - %s/%s\n", $irn, $title, $type, $format);
    }
}
```

This will produce output similar to the following:

```
irn 100105: Signature of Luciano Pavarotti - image/jpeg
irn 100096: Luciano Pavarotti - image/gif
irn 100100: Luciano Pavarotti with Celine Dion - image/gif
irn 100101: Luciano Pavarotti with Natalie Cole - image/gif
irn 100102: Luciano Pavarotti with the Spice Girls - image/gif
```

<h3 id="3-4-2-multimedia-files">Multimedia Files</h3>

Similarly, information about the multimedia files associated with a Multimedia module record can be retrieved using the `IMu::Module` class `fetch` method by specifying one of the following [virtual columns](#3-3-2-6-virtual-columns).

The following virtual columns return information about a single multimedia file from the current Multimedia record. The information is returned as a associative array.

* master
* resource
* thumbnail

The following virtual columns return information about a set of multimedia files of the current Multimedia record. The information is returned as an array. Each array member is a associative array containing the information for a single multimedia file from the set:

* resolutions
* resources
* supplementary

The *master*, *thumbnail*, *resolutions* and *supplementary* virtual columns all return the same type of information. That information differs for image and non-image multimedia as follows:

For non-image multimedia they return:

* 
    fileSize

    The size of the file in bytes.

* 
    identifier

    The name of the multimedia file.

* 
    index

    An integer that specifies the multimedia files position in the list of the master, thumbnail, resolutions and supplementary (in that order) multimedia files numbered from 0.

* 
    kind

    The kind (master, thumbnail, resolution, or supplementary) of the multimedia.

* 
    md5Checksum

    The [MD5](http://en.wikipedia.org/wiki/MD5) checksum of the multimedia file.

* 
    md5Sum

    The same as md5Checksum (included for backwards compatibility).

* 
    mimeFormat

    The media format.

* 
    mimeType

    The media type.

* 
    size

    The same as fileSize (included for backwards compatibility).

For image multimedia they return all of the values specified for non-image multimedia and also include:

* 
    bitsPerPixel

    The colour depth of the image.

* 
    colourSpace

    The colour space of the image.

* 
    compression

    The type of compression used on the image.

* 
    height

    The height of the image in pixels.

* 
    imageType

    The type classification of the image. For example:

    * Bilevel: Specifies a monochrome image.
    
    * ColorSeparation: Specifies a grayscale image.

    * Grayscale: Specifies a grayscale image.

    * GrayscaleMatte: Specifies a grayscale image with opacity.

    * Palette: Specifies a indexed color (palette) image.

    * PaletteMatte: Specifies a idexed color (palette) image with opacity.

    * TrueColor: Specifies a truecolor image.

    * TrueColorMatte: Specifies a truecolor image with opacity.

    Some more information can be found [here](http://www.imagemagick.org/Magick++/Enumerations.html#ImageType)

* 
    numberColours

    The number of colours in the image.

* 
    numberPages

    The number of images within the main image - a feature that is supported only in certain file types, e.g. TIFF.

* 
    planes

    The number of planes in an image.

* 
    quality

    An integer value from 1 to 100 that indicates the quality of the image. A lower value indicates a lower image quality and higher compression and a higher value indicates a higher image quality but a lower compression. Only applicable to JPEG and MPEG image formats.

* 
    resolution

    The resolution of the image in <abbr title="Pixels per inch">PPI</abbr>.

* 
    width

    The width of the image in pixels.

The *resource* and *resources* virtual columns both return the same type of information as follows:

* 
    identifier

    The name of the multimedia file.

* 
    mimeType

    The media type.

* 
    mimeFormat

    The media format.

* 
    size

    The size of the file in bytes.

* 
    file

    An [IO:File](http://perldoc.perl.org/IO/File.html) object. This provides a read-only handle to a temporary copy of the multimedia file. The temporary copy of the file is discarded when the handle is closed or destroyed.

> **NOTE:**
>
> If the resource column is specified with a filter, a modifier must also be provided in order for the file handle to be returned, eg:
>
> ```
> 'resources(height @ 200){resource:include}'
> ```
> 
> Modifier options include:
>
> 1. resource:include - (will add the file handle to the data set returned)
> 1. resource:only - (will replace the data set returned with the file handle)

* 
    height

    The height of the image in pixels.

* 
    width

    The width of the image in pixels.

<h4 id="3-4-2-1-example">Example</h4>

This example shows the retrieval of the multimedia title and resource information about all multimedia files for all multimedia images attached to a parties record:

```
my $columns =
[
    'images.(MulTitle,resources)',
];
my $result = $parties->fetch('start', 0, 1, $columns);
foreach my $row (@{$result->{'rows'}})
{
    foreach my $image (@{$row->{'images'}})
    {
        my $irn = $image->{'irn'};
        my $type = $image->{'type'};
        my $format = $image->{'format'};
        my $title = $image->{'MulTitle'};

        printf("irn %d: %s - %s/%s\n", $irn, $title, $type, $format);

        foreach my $resource (@{$image->{'resources'}})
        {
            my $height = $resource->{'height'};
            my $identifier = $resource->{'identifier'};
            my $mimeFormat = $resource->{'mimeFormat'};
            my $mimeType = $resource->{'mimeType'};
            my $size = $resource->{'size'};
            my $width = $resource->{'width'};

            printf("  %s: %s/%s - %dx%d - %d bytes\n", $identifier, $mimeType,
                $mimeFormat, $height, $width, $size);
        }
        print("\n");
    }
}
```

This will produce output similar to the following:

```
irn 100105: Signature of Luciano Pavarotti - image/jpeg
  signature.jpg: image/jpeg - 85x300 - 6535 bytes
  signature.thumb.jpg: image/jpeg - 25x90 - 1127 bytes

irn 100096: Luciano Pavarotti - image/gif
  LucianoPavarotti.gif: image/gif - 400x273 - 19931 bytes
  LucianoPavarotti.thumb.jpg: image/jpeg - 90x61 - 1354 bytes
  LucianoPavarotti.300x300.jpg: image/jpeg - 300x205 - 41287 bytes

irn 100100: Luciano Pavarotti with Celine Dion - image/gif
  PavarottiWithCelineDion.gif: image/gif - 400x381 - 66682 bytes
  PavarottiWithCelineDion.thumb.jpg: image/jpeg - 90x85 - 2393 bytes
  PavarottiWithCelineDion.300x300.jpg: image/jpeg - 300x286 - 76091 bytes

irn 100101: Luciano Pavarotti with Natalie Cole - image/gif
  PavarottiWithNatalieCole.gif: image/gif - 251x400 - 44551 bytes
  PavarottiWithNatalieCole.thumb.jpg: image/jpeg - 56x90 - 1768 bytes
  PavarottiWithNatalieCole.300x300.jpg: image/jpeg - 188x300 - 49698 bytes

irn 100102: Luciano Pavarotti with the Spice Girls - image/gif
  PavarottiWithSpiceGirls.gif: image/gif - 326x400 - 64703 bytes
  PavarottiWithSpiceGirls.thumb.jpg: image/jpeg - 73x90 - 2294 bytes
  PavarottiWithSpiceGirls.300x300.jpg: image/jpeg - 245x300 - 65370 bytes
```

The actual bytes of the multimedia file can be accessed using the `IO:File` object from the *file* value returned using the `resource` or `resources` virtual columns. We can use the `IO:File` to copy the file from the IMu server:

```
my $columns =
[
    'image.resource',
];

my $result = $parties->fetch('start', 0, 1, $columns);
foreach my $row (@{$result->{'rows'}})
{
    my $image = $row->{'image'};
    my $resource = $image->{'resource'};

    my $name = $resource->{'identifier'};
    my $temp = $resource->{'file'};

    my $copy = IO::File->new($name, '>');
    my $buffer;
    my $offset = 0;
    while (1)
    {
         # read 4K at a time
         my $length = $temp->sysread($buffer, 4096, $offset);
         if ($length == 0)
         {
            last;
         }
         $copy->syswrite($buffer, $length, $offset);
         $offset += $length;
     }
     $copy->close();
}
```

This copies the multimedia file from the IMu server to a local file with the same name, in this case *signature.jpg*

<h3 id="3-4-3-filters">Filters</h3>

While the Multimedia module virtual columns provide a reasonably fine-grained method for selecting specific multimedia files associated with a multimedia record, in some circumstances it is useful to have even more control over the selection of multimedia files, particularly when specifying the *resolutions*, *resources* or *supplementary* virtual columns.

Filters provide a mechanism to specify particular files associated with a multimedia record based on certain characteristics of the files. Filters consist of three parts; a name, a operator and a value. They are specified in round brackets after a virtual column:

```
column(name operator value);
```

Multiple values can be specified by separating each filter with a comma:

```
column(name operator value, name operator value);
```

*
    name

    The filter name specifies the characteristic of the multimedia file to filter on. Unless noted otherwise the meaning of the filter names is as specified in [Multimedia Files](#3-4-2-multimedia-files) section.

    The following filter names can be used to filter any multimedia file:

    * fileSize (or size)
    * height
    * identifier
    * index
    * kind
    * mimeFormat (or format)
    * mimeType (or type)
    * width

    The following filter names can be used to filter multimedia image files:

    * bitsPerPixel
    * colourSpace
    * compression
    * imageType
    * md5Checksum (or md5sum)
    * numberColours
    * numberPages
    * planes
    * quality
    * resolution

    The following filter name can be used to filter supplementary multimedia files:

    * usage

        The value of the supplementary attributes usage (SupUsage_tab) column.

*
    operator

    The operator specifies how the filter value should relate to the multimedia file characteristic specified by the filter *name*. The available values are:

    * == 

    Equals.

    Selects the multimedia files where the characteristic specified by the filter *name* is the same as the filter *value*.

    * !=

    Not equals.

    Selects the multimedia files where the characteristic specified by the filter *name* is not the same as the filter *value*.

    * <

    Less than.

    Selects the multimedia files where the characteristic specified by the filter *name* is less than the filter *value*. Only applies to numeric values.

    * \>

    Greater than.

    Selects the multimedia files where the characteristic specified by the filter *name* is greater than the filter *value*. Only applies to numeric values.

    * <=

    Less than or equal to.

    Selects the multimedia files where the characteristic specified by the filter *name* is less that or equal to the filter *value*. Only applies to numeric values.

    * \>=

    Greater than or equal to.

    Selects the multimedia files where the characteristic specified by the filter *name* is greater that or equal to the filter *value*. Only applies to numeric values.

    * @

    Closest to (also called best fit).

    Selects the single multimedia file where the characteristic specified by the filter *name* is closest to the filter *value*. Only applies to numeric values.

    * ^

    Closest to but greater than.

    Selects the single multimedia file where the characteristic specified by the filter *name* is closest to but greater than the filter *value*. Only applies to numeric values.

* 
    value

    The value to filter by. Any value can be used but, obviously, only certain values make sense for each filter. For example, if the *fileSize* filter is being used then only a numeric *value* is useful. Similarly, if the *mimeType* filter is being used then only a text *value* that corresponds to a valid MIME type is useful.

For example:

1.
    Select multimedia resolutions with a width greater that 300 pixels:

    ```
    resolutions(width > 300)
    ```

1.
    Select the single multimedia resource with a width closest to 600:

    ```
    resources(width @ 600)
    ```

1.
    Select the thumbnail resource:

    ```
    resources(kind == thumbnail)
    ```

1.
    Specify multiple filters to select the single multimedia resource with a width and height closest to 600:

    ```
    resources(width @ 600, height @ 600)
    ```

<h4 id="3-4-3-1-example">Example</h4>

This example shows the retrieval of the multimedia title and resource information about the single multimedia file with a width closest to 300 for all multimedia images attached to a parties record:

```
my $columns =
[
    'images.(MulTitle,resources(width @ 300))',
];
my $result = $parties->fetch('start', 0, 1, $columns);
foreach my $row (@{$result->{'rows'}})
{
    foreach my $image (@{$row->{'images'}})
    {
        my $irn = $image->{'irn'};
        my $type = $image->{'type'};
        my $format = $image->{'format'};
        my $title = $image->{'MulTitle'};

        printf("irn %d: %s - %s/%s\n", $irn, $title, $type, $format);

        foreach my $resource (@{$image->{'resources'}})
        {
            my $height = $resource->{'height'};
            my $identifier = $resource->{'identifier'};
            my $mimeFormat = $resource->{'mimeFormat'};
            my $mimeType = $resource->{'mimeType'};
            my $size = $resource->{'size'};
            my $width = $resource->{'width'};

            printf("  %s: %s/%s - %dx%d - %d bytes\n", $identifier, $mimeType,
                $mimeFormat, $height, $width, $size);
        }
        print("\n");
    }
}
```

> **NOTE:**
>
> The only difference from the previous example is the inclusion of a filter on the *resources* Multimedia virtual column.

This will produce output similar to the following:

```
irn 100105: Signature of Luciano Pavarotti - image/jpeg
  signature.jpg: image/jpeg - 85x300 - 6535 bytes

irn 100096: Luciano Pavarotti - image/gif
  LucianoPavarotti.gif: image/gif - 400x273 - 19931 bytes

irn 100100: Luciano Pavarotti with Celine Dion - image/gif
  PavarottiWithCelineDion.300x300.jpg: image/jpeg - 300x286 - 76091 bytes

irn 100101: Luciano Pavarotti with Natalie Cole - image/gif
  PavarottiWithNatalieCole.300x300.jpg: image/jpeg - 188x300 - 49698 bytes

irn 100102: Luciano Pavarotti with the Spice Girls - image/gif
  PavarottiWithSpiceGirls.300x300.jpg: image/jpeg - 245x300 - 65370 bytes
```

<h3 id="3-4-4-modifiers">Modifiers</h3>

While the IMu API provides a number of ways to select particular multimedia files from a Multimedia record sometimes none of the available files fulfill the required characteristics. Sometimes it is necessary to modify an existing multimedia file to achieve the desired result.

Modifiers provide a mechanism to convert multimedia images returned by the IMu server in a number of ways. The modifications are performed on-the-fly and do **not** affect the multimedia stored in the Multimedia database; they only apply to the temporary copy of multimedia returned by the IMU API. Modifiers consist of two parts; a name and a value seperated by a colon. They are specified in braces (curly brackets) after a *resource* or *resources* virtual column:

```
column{name:value}
```

Multiple values can be specified by seperating each filter with a comma:

```
column(…){name:value}
```

The supported values for name are:

* 
    checksum

    Include a checksum value with the resource (or resources) virtual column response. While this does not actually apply any modifications to a multimedia file it is useful when you require a checksum for multimedia that has had a modifier applied to it (cf. original multimedia).

    The allowed value parts are:

    * crc32

    Include a [CRC32](http://en.wikipedia.org/wiki/Cyclic_redundancy_check) checksum.

    * md5

    Include a [MD5](http://en.wikipedia.org/wiki/Md5) checksum.

    When the checksum modifier is used the resource (or resources) virtual column response includes a checksum component.

* 
    format

    Specifies that the multimedia file should be converted to the specified [format](GLOSSARY.md#mime-format). If the multimedia is not already in the required format it is reformatted on-the-fly.

    The IMu server uses ImageMagick to process the image and the range of supported formats is very large. The complete list is available from: http://www.imagemagick.org/script/formats.php. Any of the supported formats can be used as the value part of this modifier.

* 
    resource

    Specifies that a multimedia file handle should be returned.

    The allowed value parts are:

    * include
    * only

* 
    height

    Specifies that the multimedia image file should be converted to the specified height (in pixels). If the Multimedia record contains a resolution with this height, this resolution is returned instead (i.e. no modification is applied). Otherwise the closest matching larger resolution is resized to the requested height on-the-fly.

    The allowed *value* parts are any numeric value specifying the height in pixels.

* 
    width

    Specifies that the multimedia image file should be converted to the specified width (in pixels). If the Multimedia record contains a resolution with this width, this resolution is returned instead (i.e. no modification is applied). Otherwise the closest matching larger resolution is resized to the requested width on-the-fly.

    The allowed *value* parts are any numeric value specifying the width in pixels.

* 
    aspectratio

    Controls whether the image’s aspect ratio should be maintained when both a height and a width modifier are specified. If set to no, the aspect ratio is not maintained. by default the aspect ratio is maintained.

    The allowed value parts are:

    * yes
    * no

> **NOTE:**
>
> Modifiers currently only apply to multimedia images and can only be specified after the Multimedia virtual *resource* or *resources* columns.
>
> Only the *resource* or *resources* parts of the returned results are affected by modifiers. By design, all other response parts include the information for the original, unmodified multimedia.

For example:

1. Specify a Base64 encoding modifier:

```
resource{encoding:base64}
```

1. Include a CRC32 checksum in the response:

```
resource{checksum:crc32}
```

1. Reformat the multimedia image to the gif format:

```
resource{format:gif}
```

1. Resize the multimedia image to a height of 300 pixels:

```
resource{height:300}
```

1. Resize the multimedia image to a width of 300 pixels:

```
resource{width:300}
```

1. Resize the multimedia image to a height & width of 300 pixels and do not maintain aspect ratio:

```
resource{height:300, width:300, aspectratio:no}
```

#### Performance Issues

Modifying a multimedia file is computationally expensive, it should only be used when absolutely necessary. For example, it is better to use the filtering mechanism to select multimedia image files of the desired dimensions rather than modifying them to fit:

Good:

```
resource(height @ 300, width @ 300)
```

Not so good:

```
resource{height:300, width:300)
```

Obviously this only works if you have image file resolutions that are close to the desired dimensions.

Modifying a multimedia image file that is closer to the desired dimensions is less computationally expensive than modifying a larger image, so selecting the appropriate image prior to modification is preferable:

Good:

```
resource(height ^ 299, width ^ 299){height:300, width:300}
```

Not so good:

```
resource{height:300, width:300}
```

#### Example

This example shows the retrieval of the multimedia title and setting a *format*, *width* & *height* modifier to the resource information for the master multimedia image attached to a narratives record:

```
my $terms = IMu::Terms->new();
$terms->add('NarTitle', 'Angus Young');

my $hits = $narratives->findTerms($terms);

my $columns =
[
    'image.(MulTitle, resource{format:jpeg, height:600, width:600, checksum:md5})',
];
my $result = $narratives->fetch('start', 0, 1, $columns);

foreach my $row (@{$result->{'rows'}})
{
    my $image = $row->{'image'};
    my $irn = $image->{'irn'};
    my $type = $image->{'type'};
    my $format = $image->{'format'};
    my $title = $image->{'MulTitle'};

    printf("irn %d: %s - %s/%s\n", $irn, $title, $type, $format);

    my $resource = $image->{'resource'};
    my $height = $resource->{'height'};
    my $identifier = $resource->{'identifier'};
    my $mimeFormat = $resource->{'mimeFormat'};
    my $mimeType = $resource->{'mimeType'};
    my $size = $resource->{'size'};
    my $width = $resource->{'width'};
    my $checksum = $resource->{'checksum'};

    printf("  %s: %s/%s - %dx%d - %d bytes - %s\n", $identifier, $mimeType,
        $mimeFormat, $height, $width, $size, $checksum);
}
```

This will produce output similar to the following:

```
irn 165: Angus Young, AC/DC Jacket - image/tiff
  00033320.jpeg: image/jpeg - 599x401 - 77396 bytes - d94a9f46bd6274bcd20154bc513cf61f
```

The bytes of the modified multimedia can be accessed in the usual way via the *resource* response file value.

> **NOTE:**
>
> * Only the *resource* response part has been affected by the modifier. The *image* response part still reports the *format* as *tiff*. This is by design.
>
> * Because the aspect ratio has been maintained the image does not have the exact height and width specified.

<h1 id="4-maintaining-state">Maintaining State</h1>

One of the biggest drawbacks of the [earlier example](#3-3-3-example) is that it fetches the full set of results at one time, which is impractical for large result sets. It is more practical to display a full set of results across multiple pages and allow the user to move forward or backward through the pages.

This is simple in a conventional application where a connection to the separate server is maintained until the user terminates the application. In a web implementation however, this seemingly simple requirement involves a considerably higher level of complexity due to the stateless nature of web pages. One such complexity is that each time a new page of results is displayed, the initial search for the records must be re-executed. This is inconvenient for the web programmer and potentially slow for the user.

The IMu server provides a solution to this. When a handler object is created, a corresponding object is created on the server to service the handler’s request: this server-side object is allocated a unique identifier by the IMu server. When making a request for more information, the unique identifier can be used to connect a new handler to the same server-side object, with its state intact.

The following example illustrates the connection of a second, independently created `IMu::Module` object to the same server-side object:

```
# Create a module object as usual
my $first = IMu::Module->new('eparties', $session);

# Run a search - this will create a server-side object
my $keys = [1, 2, 3, 4, 5, 42];
$first->findKeys($keys);

# Get a set of results
my $result1 = $first->fetch('start', 0, 2, 'SummaryData');

# Create a second module object
my $second = IMu::Module->new('eparties', $session);

# Attach it to the same server-side object as the
# first module. This is the key step.
$second->{'id'} = $first->{'id'};

# Get a second set of results from the same search
my $result2 = $second->fetch('current', 1, 2, 'SummaryData');
```

Although two completely separate `IMu::Module` objects have been created, they are each connected to the same server-side object by virtue of having the same `id` property. This means that the second `fetch` call will access the same result set as the first `fetch`. Notice that a flag of *current* has been passed to the second call. The current state is maintained on the server-side object, so in this case the second call to `fetch` will return the third and fourth records in the result set.

While this example illustrates the use of the `id` property, it is not particularly realistic as it is unlikely that two distinct objects which refer to the same server-side object would be required in the same piece of code. The need to re-connect to the same server-side object when generating another page of results is far more likely. This situation involves creating a server-side `IMu::Module` object (to search the module and deliver the first set of results) in one request and then re-connecting to the same server-side object (to fetch a second set of results) in a second request. As before, this is achieved by assigning the same identifier to the `id` property of the object in the second page, but two other things need to be considered.

By default the IMu server destroys all server-side objects when a session finishes. This means that unless the server is explicitly instructed not to do so, the server-side object may be destroyed when the connection from the first page is closed. Telling the server to maintain the server-side object only requires that the `destroy` property on the object is set to false before calling any of its methods. In the example above, the server would be instructed not to destroy the object as follows:

```
my $module = IMu::Module->new('eparties', $session);
$module->setDestroy(0);
my $keys = [1, 2, 3, 4, 5, 42];
$module->findKeys($keys);
```

The second point is quite subtle. When a connection is established to a server, it is necessary to specify the port to connect to. Depending on how the server has been configured, there may be more than one server process listening for connections on this port. Your program has no control over which of these processes will actually accept the connection and handle requests. Normally this makes no difference, but when trying to maintain state by re-connecting to a pre-existing server-side object, it is a problem.

For example, suppose there are three separate server processes listening for connections. When the first request is executed it connects, effectively at random, to the first process. This process responds to the request, creates a server-side object, searches the Parties module for the terms provided and returns the first set of results. The server is told not to destroy the object and passes the server-side identifier to another page which fetches the next set of results from the same search.

The problem comes when the next page connects to the server again. When the connection is established any one of the three server processes may accept the connection. However, only the first process is maintaining the relevant server-side object. If the second or third process accepts the connection, the object will not be found.

The solution to this problem is relatively straightforward. Before the first request closes the connection to its server, it must notify the server that subsequent requests need to connect explicitly to that process. This is achieved by setting the `IMu::Session` object’s `suspend` property to *true* prior to submitting any request to the server:

```
my $session = IMu::Session->new('server.com', 12345);
my $module = IMu::Module->new('eparties', $session);
$session->setSuspend(1);
$module->findKeys($keys);
```

The server handles a request to suspend a connection by starting to listen for connections on a second port. Unlike the primary port, this port is guaranteed to be used only by that particular server process. This means that a subsequent page can reconnect to a server on this second port and be guaranteed of connecting to the same server process. This in turn means that any saved server-side object will be accessible via its identifier. After the request has returned (in this example it was a call to `findKeys`), the `IMu::Session` object’s `port` property holds the port number to reconnect to:

```
$session->setSuspend(1);
$module->findKeys($keys);
$reconnect = $session->{'port'};
```

<h2 id="4-1-example">Example</h2>

To illustrate we’ll modify the very simple results page of the [earlier section](#3-3-3-example) to display the list of matching names in blocks of five records per page. We’ll provide simple *Next* and *Prev* links to allow the user to move through the results, and we will use some more `GET` parameters to pass the port we want to reconnect to, the identifier of the server-side object and the rownum of the first record to be displayed.

First build the search page, which is a plain HTML form:

```
<head>
  <title>Party Search</title>
</head>
<body>
  <form action="example.php">
    <p>Enter a last name to search for (e.g. S*):</p>
    <input type="text" name="name"/>
    <input type="submit" value="Search"/>
  </form>
</body>
```

Next build the results page, which runs the search and displays the results. The steps to build the search page are outlined in detail below.

1. 
    Create an `IMu::Session` object. Then the `port` property is set to a standard value unless a *port* parameter has been passed in the URL.

    ```
    #
    # Create new session object.
    #
    my $session = IMu::Session->new();
    $session->setHost('imu.mel.kesoftware.com');

    #
    # Work out what port to connect to
    #
    my $port = 40136;
    if (defined($cgi->param('port')) && $cgi->param('port') ne '')
    {
        $port = $cgi->param('port')
    }
    $session->setPort($port);
    ```

1. 
    Connect to the server and immediately set the suspend property to *1* (true) to tell the server that we may want to connect again:

    ```
    #
    # Establish connection and tell the server we may want to re-connect
    #
    $session->connect();
    $session->setSuspend(1);
    ```

    This ensures the server listens on a new, unique port.

1. 
    Create the client-side `IMu::Module` object and set its destroy property to *0* (false):

    ```
    #
    # Create module object and tell the server not to destroy it.
    #
    my $module = IMu::Module->new('eparties', $session);
    $module->setDestroy(0);
    ```

    This ensures that the server will not destroy the corresponding server-side object when the session ends.

1. 
    If the URL included a *name* parameter, we need to do a new search. Alternatively, if it included an *id* parameter, we need to connect to an existing server-side object:

    ```
    #
    # If name is supplied, do new search.
    #
    my $name = $cgi->param('name');
    my $id = $cgi->param('id');

    if (defined($name) && $name ne '')
    {
        my $terms = IMu::Terms->new();
        $terms->add('NamLast', $name);
        $module->findTerms($terms);
    }
    #
    # Otherwise, if id is supplied reattach to existing server-side object
    #
    elsif (defined($id) && $id ne '')
    {
        $module->{'id'} = $id;
    }
    #
    # Otherwise, we can't process
    #
    else
    {
        print($cgi->header('text/html','400 Bad Request'));
        die("missing 'name' or 'id' parameter\n");
    }
    ```

1. 
    Build a list of columns to fetch:

    ```
    my $columns =
    [
        'NamFirst',
        'NamLast'
    ];
    ```

1. 
    If the URL included a *rownum* parameter, fetch records starting from there. Otherwise start from record number *1*:

    ```
    #
    # Work out which block of records to fetch
    #
    my $rownum = 1;
    if (defined($cgi->param('rownum')))
    {
        $rownum = $cgi->param('rownum');
    }
    ```

1. 
    Build the main page:

    ```
    #
    # Fetch next five records
    #
    my $results = $module->fetch('start', $rownum - 1, 5, $columns);
    my $hits = $results->{'hits'};

    #
    # Build the results page
    #
    print($cgi->header());
    print($cgi->start_html(-title => 'IMu Perl API - Maintaining State'));       
    print($cgi->p("Number of matches: $hits"));

    #
    # Display each match in a separate row in a table
    #
    print($cgi->start_table());

    foreach my $row (@{$results->{'rows'}})
    {
        my $rowNum = $row->{'rownum'};
        my $firstName = $row->{'NamFirst'};
        my $lastName = $row->{'NamLast'};

        print($cgi->start_Tr()); 
        print($cgi->start_td(), "$rowNum.", $cgi->end_td());
        print($cgi->start_td(), "$lastName, $firstName", $cgi->end_td());
        print($cgi->end_Tr());
    }

    print($cgi->end_table(), "\n");
    ```

1. 
    Finally, add the *Prev* and *Next* links to allow the user to page backwards and forwards through the results. This is the most complicated part! First, to ensure that a connection is made to the same server and server-side object, add the appropriate *port* and *id* parameters to the link URL:

    ```
    #
    # Add the Prev and Next links
    #
    my $url = $cgi->url();
    $url .= '?port=' . $session->{'port'};
    $url .= '&id=' . $module->{'id'};
    ```

1. 
    If the first record is not showing add a Prev link to allow the user to go back one page in the result set. Similarly, if the last record is not showing add a *Next* link to allow the user to go forward one page:

    ```
    my $first = $results->{'rows'}->[0];
    if ($first->{'rownum'} > 1)
    {
        my $prev = $first->{'rownum'} - 5;
        if ($prev < 1)
        {
            $prev = 1;
        }
        $prev = $url . '&rownum=' . $prev;
        print($cgi->a({href => $prev}, 'Prev'), "\n");
    }

    my $count = @{$results->{'rows'}}; 
    my $last = $results->{'rows'}->[$count - 1];
    if ($last->{'rownum'} < $results->{'hits'})
    {
        my $next = $last->{'rownum'} + 1;
        $next = $url . '&rownum=' . $next;
        print($cgi->a({href => $next}, 'Next'));
    }
    print($cgi->end_html());
    exit(0);
    ```

<h1 id="5-logging-in-to-an-imu-server">Logging in to an IMu server</h1>

When an IMu based program connects to an IMu server it is given a default level of access to EMu modules.

It is possible for an IMu based program to override this level of access by explicitly logging in to the server as a registered user of EMu. This is done by using the `IMu::Session‘s` `login` method. Once the `login` method has been called successfully the session remains authenticated until the `logout` method is called.

<h2 id="5-1-the-login-method">The login method</h2>

The login method is used to authenticate the program as a registered user of EMu. Once successfully authenticated access to EMu modules is at the level of the authenticated user rather than the default imuserver user.

<h3 id="5-1-1-arguments">Arguments</h3>

* 
    username

    The name of the user to login as. This must be the name of a registered EMu user on the system.

* 
    password

    The user’s password. This argument is optional and if it is not supplied it defaults to `null`.

    > **NOTE:**
    >
    > Supplying a `null` password is uncommon but it is sometimes a valid thing to do. If the server receives a password of `null` it will try to authenticate the user using server-side methods such as verification against emu’s .rhosts file.

* 
    spawn

    A boolean value indicating whether the IMu server should create a separate process dedicated to handling this program’s requests. This argument is optional and if not supplied it defaults to `true`.

<h2 id="5-2-the-logout-method">The logout method</h2>

The logout method relinquishes access as the previously authenticated user.

> **NOTE:**
>
> Logging in this way is very similar to logging into the same EMu environment using the EMu client. Access to records is controlled via record-level security.

> **WARNING:**
>
> Logging in causes the IMu server to start a new texserver process to handle all access to EMu module. This new texserver process will use a Texpress licence. The licence will not be freed until the logout method is called. See the server FAQ [How does IMu use Texpress licences?](FAQ.md#-How-does-imu-use-texpress-licences) for more information.

<h1 id="6-updating-an-emu-module">Updating an EMu Module</h1>

The `IMu::Module` class provides methods for inserting new records and for updating or removing existing records in any EMu module.

> **NOTE:**
>
> By default these operations are restricted by the IMu server. Typically access to these operations is gained by [logging in to the IMu server](#5-1-the-login-method). See the [allow-updates](CONFIGURATION.md#allow-updates) entry of the server configuration for more information.

<h2 id="6-1-the-insert-method">The insert Method</h2>

The `insert` method is used to add a new record to the module.

<h3 id="6-1-1-arguments">Arguments</h3>

The method takes two arguments:

* 
    values

    The *values* argument specifes any data values to be inserted into the newly created record.

    The *values* should be an associative array. The indexes of the array must be column names.

* 
    columns

    The *columns* argument is used to specify which columns should be returned once the record has been created. The value of the *column* is specified in exactly the same way as in the `fetch` method. See the section on [Specifying Columns](#3-3-2-specifying-columns) for details.

    > **NOTE:**
    >
    > It is very common to include `irn` as one of the columns to be returned. This gives a way of getting the key of the newly created record.

<h3 id="6-1-2-return-value">Return Value</h3>

The method returns a [Map](todo link to reference) object. The `Map` contains an entry for each column requested.

<h3 id="6-1-3-example">Example</h3>

```
my $parties = IMu::Module->new('eparties', $session);

# 
# Specify the values to insert.
#
my $values =
{
    NamFirst => 'Chris',
    NamLast => 'Froome',
    NamOtherNames_tab =>
    [
        'Christopher',
        'Froomey'
    ]
};

# 
# Specify the column values to return after inserting.
#
my $columns =
[
    'irn',
    'NamFirst',
    'NamLast',
    'NamOtherNames_tab'
];

# 
# Insert the new record.
#
my $result;
eval
{
    $result = $parties->insert($values, $columns);
};
if ($@)
{
    die("Error: $@\n");
}

#
# Output the returned values.
#
my $irn = $result->{irn};
my $first = $result->{NamFirst};
my $last = $result->{NamLast};
my $others = $result->{NamOtherNames_tab};

print("$last, $first ($irn)\n");
print("Other names:\n");
foreach my $other (@$others)
{
    print("\t$other\n");
}
exit(0);
```

If inserting of records is permitted this will produce output similar to the following:

```
Froome, Chris (435)
Other names:
	Christopher
	Froomey
```

If inserting of records is denied by the server this will produce output similar to the following:

```
Error: ModuleUpdatesNotAllowed (authenticated,default)
```

<h2 id="6-2-the-update-method">The update Method</h2>

The `update` method is used to modify one or more existing records. This method operates very similarly to the `fetch` method. The only difference is a *values* argument which contains a set of values to be applied to each specified record.

<h3 id="6-2-1-arguments">Arguments</h3>

The method takes five arguments:

* **flag**
* **offset**
* **count**

    These arguments are identical to those used by the [fetch](#3-3-1-the-fetch-method) method. They define the starting position and size of the block of records to be updated.

* **values**

    The _values_ argument specifies the columns to be updated in the specified block of records. The _values_ argument must be a hash reference. The keys of the hash must be column names.

    This is the same as the values argument for the [insert](#6-1-the-insert-method) method.

* **columns**

    The *columns* argument is used to specify which columns should be returned once the record has been created. The value of the *column* is specified in exactly the same way as in the `fetch` method. See the section on [Specifying Columns](#3-3-2-specifying-columns) for details.

    This is the same as the *columns* argument for the `insert` method.

<h3 id="6-2-2-return-value">Return Value</h3>

The `update` method returns an `IMuModuleFetchResult` object (the same
as the [fetch](#3-3-1-the-fetch-method) method). It contains
the values for the selected block of records after the updates have been
applied.

<h3 id="6-2-3-example">Example</h3>

```
#
# Find all parties records that have a first name of "Chris" and a last name of
# "Froome".
#
my $parties = IMu::Module->new('eparties', $session);
my $terms = IMu::Terms->new();
$terms->add('NamFirst', 'Chris');
$terms->add('NamLast', 'Froome');
$parties->findTerms($terms);

#
# Specify the column to update and the new value.
#
my $values =
{
    NamFirst => 'Christopher'
};

# 
# Specify the column values to return after updating.
#
my $columns =
[
    'irn',
    'NamFirst',
    'NamLast'
];

# 
# Run the update.
#
my $result;
eval
{
    $result = $parties->update('start', 0, -1, $values, $columns);
};
if ($@)
{
    die("Error: $@\n");
}

# 
# Output the returned values.
#
print("Count: $result->{count}\n");
print("Hits: $result->{hits}\n");

print("Rows:\n");
foreach my $row (@{$result->{rows}})
{
    my $rowNum = $row->{rownum};
    my $irn = $row->{irn};
    my $first = $row->{NamFirst};
    my $last = $row->{NamLast};

    print("\t$rowNum. $last, $first ($irn)\n");
}
exit(0);
```

If updating records is allowed the example will produce output similar to the following:

```
Count: 1
Hits: 1
Rows:
	1. Froome, Christopher (435)
```

<h2 id="6-3-the-remove-method">The remove Method</h2>

The `remove` method is used to remove one or more existing records.

<h3 id="6-3-1-arguments">Arguments</h3>

The method takes three arguments:

* **flag**
* **offset**
* **count**

These arguments define the starting position and size of the block of records to be removed. They are identical to those used by the [fetch](TODO-link-to-reference) and [update](TODO-link-to-reference) methods.

<h3 id="6-3-2-return-value">Return Value</h3>

The method returns a `long` that specifies the number of records that were removed.

<h3 id="6-3-3-example">Example</h3>

```
# 
# Find all parties records that have a first name of "Christopher" and a last
# name of "Froome".
#
my $parties = IMu::Module->new('eparties', $session);
my $terms = IMu::Terms->new();
$terms->add('NamFirst', 'Christopher');
$terms->add('NamLast', 'Froome');
$parties->findTerms($terms);

# 
# Remove all of the matching records.
#
my $result;
eval
{
    $result = $parties->remove('start', 0, -1);
};
if ($@)
{
    die("Error: $@\n");
}
print("Removed $result record(s)\n");
exit(0);
```

If removing records is allowed the example will produce output similar to the following:

```
Removed 1 record(s)
```

<h1 id="7-exceptions">Exceptions</h1>

When an error occurs, the IMu Perl API throws an exception (`die` in Perl terminology). The exception is an `IMu::Exception` object.

For simple error handling all that is usually required is to catch the exception and report the exception as a string:

```
eval
{
    # ⋯
};
if ($@)
{
    die("Error: $@");
}
```

The `IMu::Exception` class overrides the Perl `""` (stringify) operator (which is called "magically" when the exception object is used as a string) and returns an error message.

To handle specific IMu errors it is necessary to check the exception is an `IMu::Exception` object before using it. The `IMu::Exception` class includes a property called `id`. This is a string and contains the internal IMu error code for the exception. For example, you may want to catch the exception raised when an `IMu::Session` objects `connect` method fails and try to connect to an alternative server:

```
my $mainServer = 'server1.com';
my $alternativeServer = 'server2.com';
my $session = IMu::Session->new();
$session->{'host'} = $mainServer;
eval
{
    $session->connect();
};
if ($@)
{
    if (ref($@) && $@->isa('IMu::Exception'))
    {
        # Check for specific SessionConnect error
        if ($@->getID() eq 'SessionConnect')
        {
            $session->{'host'} = $alternativeServer;
            eval
            {
                $session->connect();
            };
        }
    }
    # Check for exception again. If we successfully connected
    # to $alternativeServer then $@ will be cleared.
    if ($@)
    {
        die("Error: $@\n");
    }
}
# By the time we get to here the session is connected to either
# the main server or the alternative.
```
