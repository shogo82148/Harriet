# NAME

Harriet - Daemon manager for testing

# SYNOPSIS

    use Harriet;

    my $harriet = Harriet->new('t/harriet/');
    my $stf_url = $harriet->load('TEST_STF');

# DESCRIPTION

In some case, test code requires daemons like memcached, STF, or groonga.
If you are running these daemons for each test scripts, it eats lots of time.

Then, you need to keep the processes under the test suite.

Harriet solves this issue.

Harriet loads all daemons when starting [prove](http://search.cpan.org/perldoc?prove). And set the daemon's end point to the environment variable.
And run the test cases. Test script can use the daemon process (You need to clear the data if you need.).

# TUTORIAL

## Writing harriet script

harriet script is just a perl script has `.pl` extension. Example code is here:

    # t/harriet/TEST_MEMCACHED.pl
    use strict;
    use utf8;
    use Test::TCP;

    my $server = Test::TCP->new(
        code => sub {
            my $port = shift;
            exec '/usr/bin/memcached', '-p', $port;
            die $!;
        }
    );
    return ('127.0.0.1:' . $server->port, $server);

This code runs memcached. It returns memcached's end point information and guard object. Harriet keeps guard objects while perl process lives.

(Guard object is optional.)

## Load harriet script

    use Harriet;

    my $harriet = Harriet->new('t/harriet');
    my $memcached_endpoint = $harriet->load('TEST_MEMCACHED');

This script loads end point of memcached daemon. If there is `$ENV{TEST_MEMCACHED}` varaible, just return it.
Otherwise, harriet loads harriet script named 't/harriet/TEST\_MEMCACHED.pl' and it returns the return value of the script.

## Save daemon process under the prove

    # .proverc
    -PHarriet=t/harriet/

[App::Prove::Plugin::Harriet](http://search.cpan.org/perldoc?App::Prove::Plugin::Harriet) loads harriet scripts under the `t/harriet/`, and set these to environment variables.

This plugin starts daemons before running test cases!

# LICENSE

Copyright (C) tokuhirom.

This library is free software; you can redistribute it and/or modify
it under the same terms as Perl itself.

# AUTHOR

tokuhirom <tokuhirom@gmail.com>
