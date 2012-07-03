# redis-config

Redis-Config is a micro-tool designed to be used for extracting common Redic
configuration syntaxes from a common file format and making them available in
other formats.

## Example Usage

Given the following file:

    $ cat config/redis.yml
    ---
    host:      localhost
    port:      6380
    database:  1
    namespace: example

### On The Command Line

One can do this:

    $ redis-config --format arglist
    -p 6380 -h localhost -d 1

There are multiple format options available:

    $ redis-config --format dsn
    redis://localhost:6380/1/example

    $ redis-config --format json
    {host:'localhost',port:6380,database:1,namespace:'example'}

    $ redis-config --format {yaml,yml}
    ---
    host:      localhost
    port:      6380
    database:  1
    namespace: example

    $ redis-config --format xml
    <?xml version="1.0">
    <redis-config>
      <host>localhost</host>
      <port>6380</port>
      <database>1</database>
      <namespace>example</namespace>
    </redis-config>

### From Ruby

    RedisConfig.new('config/redis.yml').to_hash
    RedisConfig.new('config/redis.yml').to_xml
    RedisConfig.new('config/redis.yml').to_s

The `RedisConfig.new` is implied, so this is also possible:

    RedisConfig.to_hash # Pass to JSON.encode or YAML.encode
    RedisConfig.to_xml  # XML
    RedisConfig.to_s    # DSN

## Environments

They're not supported yet, but Rails (or Rack) style environments will also
supported, this will work something like this:


    $ cat config/redis.yml
    ---
      production:
        host:      localhost
        port:      6380
        database:  0
        namespace: production
      development:
        host:      localhost
        port:      6380
        database:  1
        namespace: development

`redis-config` will read the following varaibles, stopping and taking the
valud of the first one it finds: `REDIS_ENV`, `RACK_ENV`, `RAILS_ENV`,
otherwise the environment can be specified on the command line, or in Ruby
with the following:

    $ redis-config -e production

    $ REDIS_ENV=development redis-config

    RedisConfig.to_s(:production)
    RedisConfig.to_hash(:production)

    RedisConfig.new('config/redis.yml', :production).to_s
    RedisConfig.new('config/redis.yml', :production).to_hash

If working with the environmental format the default will be `development` in
the absense of any switches or environmental variables

## Use Case

    $ redis-cli $(redis-config)
    $ redis-cli  `redis-config`

`redis-config` in this case can determine that it's *stdin* is not a TTY and
can default to the `arglist` format.

## Rationale

Small tools that do one thing, and do it well are the cornerstones of a useful
toolbox, many, many Ruby projects and also projects in other languages
implement their own brand of redis-configuration, and they are often
incompatible or incomplete. (lack of namespace, DSN vs. hash structure, etc)
by standardising the richest format (hash) and making easy to interoperate
between them, I think this project can help.

## Supported Options

The `socket` option from Redis is unsupported, simply because I haven't needed
it, it should be trivial to replace host/port with a socket option.

The `password` option is also unsupported, simply because I've never needed it

The `namespace` option is a convenience provided by some Ruby libraries to
prefix all keys, and `scope` the database internally. This is ideal for multi-
tennant applications for example when the background job queue and the cacheing
layer both share the same datastore.

