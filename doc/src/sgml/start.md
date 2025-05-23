<!-- doc/src/sgml/start.sgml -->

<chapter id="tutorial-start">

# Getting Started

<sect1 id="tutorial-install">

## Installation

* TODO:
   <para>
    Before you can use <productname>PostgreSQL</productname> you need
    to install it, of course.  It is possible that
    <productname>PostgreSQL</productname> is already installed at your
    site, either because it was included in your operating system
    distribution or because the system administrator already installed
    it.  If that is the case, you should obtain information from the
    operating system documentation or your system administrator about
    how to access <productname>PostgreSQL</productname>.
   </para>

   <para>
    If you are not sure whether <productname>PostgreSQL</productname>
    is already available or whether you can use it for your
    experimentation then you can install it yourself.  Doing so is not
    hard and it can be a good exercise.
    <productname>PostgreSQL</productname> can be installed by any
    unprivileged user; no superuser (<systemitem>root</systemitem>)
    access is required.
   </para>

   <para>
    If you are installing <productname>PostgreSQL</productname>
    yourself, then refer to <xref linkend="installation"/>
    for instructions on installation, and return to
    this guide when the installation is complete.  Be sure to follow
    closely the section about setting up the appropriate environment
    variables.
   </para>

   <para>
    If your site administrator has not set things up in the default
    way, you might have some more work to do.  For example, if the
    database server machine is a remote machine, you will need to set
    the <envar>PGHOST</envar> environment variable to the name of the
    database server machine.  The environment variable
    <envar>PGPORT</envar> might also have to be set.  The bottom line is
    this: if you try to start an application program and it complains
    that it cannot connect to the database, you should consult your
    site administrator or, if that is you, the documentation to make
    sure that your environment is properly set up.  If you did not
    understand the preceding paragraph then read the next section.
   </para>
  </sect1>


<sect1 id="tutorial-arch">

## Architectural Fundamentals

* goal
  * PostgreSQL system architecture

* 💡PostgreSQL -- uses a -- client/server model 💡
  * PostgreSQL session ==
    * server process or `postgres` /
      * manages the database files,
      * 👀from client applicationS -- can handle MULTIPLE concurrent connections to the --  database 👀
        * 🧠-- via -- starts ( or forks) a NEW process / EACH connection 🧠
          * NEW == != original postgres process / started the communication
          * 👀-> types of server process 👀
            * supervisor server process
              * ALWAYS run
              * -- wait for -- associated ones
            * associated server processes
      * server  from clients.
      * performs database actions
    * user's client (frontend) application /
      * wants to perform database operations
      * diverse nature
        * _Example:_ text-oriented tool, a graphical application, web server, database maintenance tool, ...
      * MOST use PostgreSQL distribution
  * 👀client's host can be != server's host 👀
    * if it's != -> communicate -- via -- TCP/IP network connection  
    * ⚠️if you need to access files -> verify host | they are left ⚠️

<sect1 id="tutorial-createdb">

## Creating a Database

   <indexterm zone="tutorial-createdb">
    <primary>database</primary>
    <secondary>creating</secondary>
   </indexterm>

   <indexterm zone="tutorial-createdb">
    <primary>createdb</primary>
   </indexterm>

   <para>
    The first test to see whether you can access the database server
    is to try to create a database.  A running
    <productname>PostgreSQL</productname> server can manage many
    databases.  Typically, a separate database is used for each
    project or for each user.
   </para>

   <para>
    Possibly, your site administrator has already created a database
    for your use.  In that case you can omit this step and skip ahead
    to the next section.
   </para>

   <para>
    To create a new database from the command line, in this example named
    <literal>mydb</literal>, you use the following command:
<screen>
<prompt>$</prompt> <userinput>createdb mydb</userinput>
</screen>
    If this produces no response then this step was successful and you can skip over the
    remainder of this section.
   </para>

   <para>
    If you see a message similar to:
<screen>
createdb: command not found
</screen>
    then <productname>PostgreSQL</productname> was not installed properly.  Either it was not
    installed at all or your shell's search path was not set to include it.
    Try calling the command with an absolute path instead:
<screen>
<prompt>$</prompt> <userinput>/usr/local/pgsql/bin/createdb mydb</userinput>
</screen>
    The path at your site might be different.  Contact your site
    administrator or check the installation instructions to
    correct the situation.
   </para>

   <para>
    Another response could be this:
<screen>
createdb: error: connection to server on socket "/tmp/.s.PGSQL.5432" failed: No such file or directory
        Is the server running locally and accepting connections on that socket?
</screen>
    This means that the server was not started, or it is not listening
    where <command>createdb</command> expects to contact it.  Again, check the
    installation instructions or consult the administrator.
   </para>

   <para>
    Another response could be this:
<screen>
createdb: error: connection to server on socket "/tmp/.s.PGSQL.5432" failed: FATAL:  role "joe" does not exist
</screen>
    where your own login name is mentioned.  This will happen if the
    administrator has not created a <productname>PostgreSQL</productname> user account
    for you.  (<productname>PostgreSQL</productname> user accounts are distinct from
    operating system user accounts.)  If you are the administrator, see
    <xref linkend="user-manag"/> for help creating accounts.  You will need to
    become the operating system user under which <productname>PostgreSQL</productname>
    was installed (usually <literal>postgres</literal>) to create the first user
    account.  It could also be that you were assigned a
    <productname>PostgreSQL</productname> user name that is different from your
    operating system user name; in that case you need to use the <option>-U</option>
    switch or set the <envar>PGUSER</envar> environment variable to specify your
    <productname>PostgreSQL</productname> user name.
   </para>

   <para>
    If you have a user account but it does not have the privileges required to
    create a database, you will see the following:
<screen>
createdb: error: database creation failed: ERROR:  permission denied to create database
</screen>
    Not every user has authorization to create new databases.  If
    <productname>PostgreSQL</productname> refuses to create databases
    for you then the site administrator needs to grant you permission
    to create databases.  Consult your site administrator if this
    occurs.  If you installed <productname>PostgreSQL</productname>
    yourself then you should log in for the purposes of this tutorial
    under the user account that you started the server as.

    <footnote>
     <para>
      As an explanation for why this works:
      <productname>PostgreSQL</productname> user names are separate
      from operating system user accounts.  When you connect to a
      database, you can choose what
      <productname>PostgreSQL</productname> user name to connect as;
      if you don't, it will default to the same name as your current
      operating system account.  As it happens, there will always be a
      <productname>PostgreSQL</productname> user account that has the
      same name as the operating system user that started the server,
      and it also happens that that user always has permission to
      create databases.  Instead of logging in as that user you can
      also specify the <option>-U</option> option everywhere to select
      a <productname>PostgreSQL</productname> user name to connect as.
     </para>
    </footnote>
   </para>

   <para>
    You can also create databases with other names.
    <productname>PostgreSQL</productname> allows you to create any
    number of databases at a given site.  Database names must have an
    alphabetic first character and are limited to 63 bytes in
    length.  A convenient choice is to create a database with the same
    name as your current user name.  Many tools assume that database
    name as the default, so it can save you some typing.  To create
    that database, simply type:
<screen>
<prompt>$</prompt> <userinput>createdb</userinput>
</screen>
   </para>

   <para>
    If you do not want to use your database anymore you can remove it.
    For example, if you are the owner (creator) of the database
    <literal>mydb</literal>, you can destroy it using the following
    command:
<screen>
<prompt>$</prompt> <userinput>dropdb mydb</userinput>
</screen>
    (For this command, the database name does not default to the user
    account name.  You always need to specify it.)  This action
    physically removes all files associated with the database and
    cannot be undone, so this should only be done with a great deal of
    forethought.
   </para>

   <para>
    More about <command>createdb</command> and <command>dropdb</command> can
    be found in <xref linkend="app-createdb"/> and <xref linkend="app-dropdb"/>
    respectively.
   </para>
  </sect1>

<sect1 id="tutorial-accessdb">

## Accessing a Database

   <indexterm zone="tutorial-accessdb">
    <primary>psql</primary>
   </indexterm>

   <para>
    Once you have created a database, you can access it by:

    <itemizedlist spacing="compact" mark="bullet">
     <listitem>
      <para>
       Running the <productname>PostgreSQL</productname> interactive
       terminal program, called <application><firstterm>psql</firstterm></application>, which allows you
       to interactively enter, edit, and execute
       <acronym>SQL</acronym> commands.
      </para>
     </listitem>

     <listitem>
      <para>
       Using an existing graphical frontend tool like
       <application>pgAdmin</application> or an office suite with
       <acronym>ODBC</acronym> or <acronym>JDBC</acronym> support to create and manipulate a
       database.  These possibilities are not covered in this
       tutorial.
      </para>
     </listitem>

     <listitem>
      <para>
       Writing a custom application, using one of the several
       available language bindings.  These possibilities are discussed
       further in <xref linkend="client-interfaces"/>.
      </para>
     </listitem>
    </itemizedlist>

    You probably want to start up <command>psql</command> to try
    the examples in this tutorial.  It can be activated for the
    <literal>mydb</literal> database by typing the command:
<screen>
<prompt>$</prompt> <userinput>psql mydb</userinput>
</screen>
    If you do not supply the database name then it will default to your
    user account name.  You already discovered this scheme in the
    previous section using <command>createdb</command>.
   </para>

   <para>
    In <command>psql</command>, you will be greeted with the following
    message:
<screen>
psql (&version;)
Type "help" for help.

mydb=&gt;
</screen>
    <indexterm><primary>superuser</primary></indexterm>
    The last line could also be:
<screen>
mydb=#
</screen>
    That would mean you are a database superuser, which is most likely
    the case if you installed the <productname>PostgreSQL</productname> instance
    yourself.  Being a superuser means that you are not subject to
    access controls.  For the purposes of this tutorial that is not
    important.
   </para>

   <para>
    If you encounter problems starting <command>psql</command>
    then go back to the previous section.  The diagnostics of
    <command>createdb</command> and <command>psql</command> are
    similar, and if the former worked the latter should work as well.
   </para>

   <para>
    The last line printed out by <command>psql</command> is the
    prompt, and it indicates that <command>psql</command> is listening
    to you and that you can type <acronym>SQL</acronym> queries into a
    work space maintained by <command>psql</command>.  Try out these
    commands:
    <indexterm><primary>version</primary></indexterm>
<screen>
<prompt>mydb=&gt;</prompt> <userinput>SELECT version();</userinput>
                                         version
-------------------------------------------------------------------&zwsp;-----------------------
 PostgreSQL &version; on x86_64-pc-linux-gnu, compiled by gcc (Debian 4.9.2-10) 4.9.2, 64-bit
(1 row)

<prompt>mydb=&gt;</prompt> <userinput>SELECT current_date;</userinput>
    date
------------
 2016-01-07
(1 row)

<prompt>mydb=&gt;</prompt> <userinput>SELECT 2 + 2;</userinput>
 ?column?
----------
        4
(1 row)
</screen>
   </para>

   <para>
    The <command>psql</command> program has a number of internal
    commands that are not SQL commands.  They begin with the backslash
    character, <quote><literal>\</literal></quote>.
    For example,
    you can get help on the syntax of various
    <productname>PostgreSQL</productname> <acronym>SQL</acronym>
    commands by typing:
<screen>
<prompt>mydb=&gt;</prompt> <userinput>\h</userinput>
</screen>
   </para>

   <para>
    To get out of <command>psql</command>, type:
<screen>
<prompt>mydb=&gt;</prompt> <userinput>\q</userinput>
</screen>
    and <command>psql</command> will quit and return you to your
    command shell. (For more internal commands, type
    <literal>\?</literal> at the <command>psql</command> prompt.)  The
    full capabilities of <command>psql</command> are documented in
    <xref linkend="app-psql"/>.  In this tutorial we will not use these
    features explicitly, but you can use them yourself when it is helpful.
   </para>

  </sect1>
 </chapter>
