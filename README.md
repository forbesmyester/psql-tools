# Hacky tools for PostgreSQL

Hacky tools for PostgreSQL that makes interacting / extracting / analysis of data in PostgreSQL easier.

## psql-out

Sometimes I find I have requests like the following:

 > Run this query and give me the results [in an Excel file so I can look at them in Excel].

Also sometimes I want to:

 > Run a complicated query and get the results as [ndjson](http://ndjson.org/) so I can process them in JavaScript.

But sometimes I'm:

 > Looking at data trying to figure out if a hypothesis makes sense and want to throw things into graphs very, very quickly.

It's not cool, it's not pretty, but this is what `psql-out` is for!

If you've got your environmental variables set up (see `pgpass-env`) you can do something like the following:

    echo 'select name, position from competitors' | psql-out

And get a well formed tab separated CSV file out into STDOUT. If you want it as an ndjson file you can run

    echo 'select name, position from competitors' | psql-out -f ndjson

To get an ndjson file out. You can also get a TSV (Tab seperated CSV) out by passing `-t tsv`.

If you however have [termgraph](https://github.com/mkaz/termgraph) installed and in your $PATH you can run
    
    echo '
        select
            code,
            sum(CASE WHEN position = 1 THEN 1 ELSE 0 END) as win,
            sum(CASE WHEN position <= 3 THEN 1 ELSE 0 END) as podium
        from results
        natural join drivers
        group by code
        order by 2 desc limit 5' | psql-out -f graph -- --color {green,red}

And get the following back

![Result of above command](screenshot.png)

## pgpass-env

I found that I have files that look like the following in the root of most of my source code repositories (and in my `.gitignore` of course):

    export PGHOST="127.0.0.1"
    export PGDATABASE="my_product"
    export PGUSER="my_product"
    export PGPASSWORD="a_good_password"
    export PGPORT="5432"
    export LISTEN_PORT="4040"

However for my database administration GUI I also have a `~/.pgpass` file in my home directory with the following:

    127.0.0.1:5432:my_product:my_product:a_good_password

This is a duplication of data and is kinda ridiculous.

Enter `pgpass-env` which is a simple bash script that converts the former into the latter

The idea is to store a name above the lines in the ~/.pgpass like the following

    # local_my_product
    127.0.0.1:5432:my_product:my_product:a_good_password
    # local_another_product
    127.0.0.1:5432:another_product:another_product:a_good_password

Running just `pgpass-env` it gives you a list of possible options:

    $ pgpass-env
    Pass one of the following
     * local_my_product
     * local_another_product

But if you would pass the name of a connection it would output:

    $ . pgpass-env
    > postgres://my_product@127.0.0.1:5432/my_product

While at the same time performing the required `EXPORT PGUSER=my_product` etc. Using the preceding `.` means those environmental variables will be brought into the current environment, which is probably what you want.
