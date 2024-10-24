**The meaning of ...**
Or, how to interpret certain formatting and content. 

# Names

Regular instructions on a software product, use the product name at several placed, for example as a directory name but also as database name. This might lead to confusion and makes it hard to easily adjust a installation/configuration.

To solve this problem, my instruction use a specific format of the entity names (Parameters). In result my instructions can be easily customized, using a simple Search & Replace.

# Formating

## Character/Section formatting 

### Meaning of Bold text between the quote symbols

If the text between the " symbols is bold, it refers to the text as shown in the User Interface (UI). Together with the instruction it is intended to focus on the object and do something with it.

Example - User activity

tab on "**About phone**"

In the UI you will find exactly this text "About phone" and should be doing a tap on this text.

### Meaning of "\<something\>"

The `<` and `>` is a kind of a placehold. It indicates that at this place, you have to virtually insert what is mentioned between the two symbols. So replacing the whole `<something>` with the text, intended to be place at that space.

#### Example

##### File

`<path>/<file name>` would be in case of the `passwd` file = "/etc/passwd"

##### User activity

🪜 \<action\>

- You have to \<action\>, which brings you to the "\<title\>" settings.

In case of Android Smartphone, where in the "Settings" of the device, you are instructed to tap on the option "About phone".

🪜 tap on "**About phone**"

- You have to tap on "**About phone**", which brings you to the "About phone" settings.

### Meaning of `something`

In the context of activities on the Operation System (OS) level, the meaning is

- full section of activities and their output, as seen in the terminal session
- one or more words, a certain entity in the OS/Software or configuration of OS/Software

## Parameters

The entity names in my documents follow the format of `xx<purpose>xx` for production- and `xxbak<purpose>xx`for backup-instance.

- `xx` two characters represent the context, e.g. acronmy of the product name
- `bak` if specified, applies to the Backup Instance
- `<purpose>` are things like `directory`, `database` and so on.

It is pretty unlikely that a similar word is used within this documentation or exists as an entity in the used software.

## Interactions

Like you will find on every device with a Graphical User Interface (GUI) e.g. a Smartphone.

### User Interface (UI)

#### UI Objects

##### Text

"**Text**" in quotes and Bold, refers to any kind of text on the UI, e.g. Button, Field, ...

##### Menu

Same in regards to a menu, with the special format when it comes to Sub-Menu entries like "**File** > **Exit**", where the ">" is pointing from the upper menu entry to the sub menu entry.

#### Navigation

🪜 \<action\>

- You have to \<action\> to get to another dialog/page.

🛝 \<action\>
			
- You have to \<action\> to navigate backwards to the previous dialog/page.

#### Changes

✏️ \<action\>

- You have to do \<action\>, which results in an immediate change.

✔️✏️ \<action\>

- You have to do \<action\>, which results in a change once confirmed by you.

✔️ \<action\>

- You have to do \<action\> which confirms all changes (✔️✏️ \<action\>) above.

#### Actions

select "**\<option\>**"

- You have to click/tap on "**\<option\>**"

### Operation System (OS)

Execute command(s) with an output that does not matter.

```console
<command>
```

Execute command(s) and get output. With a command line starting with either a `# ` (root) or `$ ` (normal user).

```console
# <command>
<output of command>
```

Note: *See below section "File" to understand, when only a portion of the output is shown*

### Database Management System (DBMS)

Execution of a SQL-Statement in the interactive terminal. Example PostgreSQL.

```console
postgres=# <SQL Statement or command>
<output of postgres commands>
```

Execution of a SQL-Statement in the interactive terminal, where the result is irrelevant. Example PostgreSQL.

```console
<SQL Statement or command>
```

## File

A file and its content begins with the following line,

=== `<path>/<file name>` ===

With regards to the content itself, there are various ways to either show the full content or highlight only the relevant parts. 

### Complete content

```code
<file content>
```

### Partially shown content

The following are examples to highlight only some parts of the files content. Of cause these kinds can be combined.

Only the first part of the content (placeholder for the lines below is the `:`).

```code
<first lines>
:
```

Only the middle part of the content (placeholders for the other parts are the `:`).

```code
:
<middle lines>
:
```

Only the last part of the content (placeholder for the lines above is the `:`).

```code
:
<last lines>
```