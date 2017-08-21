    run = (cfg) ->

List all `user_database` fields in local numbers.

      design =
        _id: "_design/#{pkg.name}-user-dbs"
        language: 'application/javascript'
        views:
          all:
            map: p_fun (doc) ->
              return unless doc._id.match /^number:.*@/
              return unless doc.user_database?
              emit doc.user_database

      prov = new PouchDB cfg.provisioning, cfg.provisioning_options
      prov.get design._id
      .catch -> {}
      .then ({_rev}) ->
        design._rev = _rev if _rev?
        prov.put design
      .then ->
        prov.query "#{pkg.name}-user-dbs/all"
      .then ({rows}) ->
        for row in rows
          do (row) ->
            check_db cfg, row
            .catch (error) ->
              console.log "Database #{row.key} for #{row.id} is invalid."
              prov.get row.id
              .then (doc) ->
                prov.put doc
            .then ->

    check_db = (cfg,{key}) ->
      uri = url.resolve cfg.userdb_base, key
      db = new PouchDB uri, cfg.userdb_options

* doc._security CouchDB `_security` object for a database. See [Security Object](http://docs.couchdb.org/en/1.6.1/json-structure.html#security-object)
* doc._security.members.roles (array of strings) Roles required for a user to be considered a member (user) of the database.
* doc._security.admins.roles (array of strings) Roles required for a user to be considered an admin of the database.

* doc.members ignore
* doc.members.roles ignore

Make sure the database has `voicemail_settings`

      db.get 'voicemail_settings'

Ensure there is a valid security document.

      .then ->
        request.get url.resolve uri, '_security'
      .then (doc) ->
        should(doc).have.property 'members'
        should(doc.members).have.property 'roles'
        should(doc.members.roles).not.be.empty

Remove outdated voicemail messages

      .then ->
        TBD

Compact DB.

      .then ->
        db.compact()

Cleanup views.

      .then ->
        request.post url.resolve uri, '_view_cleanup'
        .accept 'json'


    pkg = require './package.json'
    cfg = require './local/config.json'
    {p_fun} = require 'coffeescript-helpers'
    PouchDB = require 'pouchdb'
    request = require 'superagent-as-promised'
    url = require 'url'
    should = (require 'should').noConflict()
    run cfg
