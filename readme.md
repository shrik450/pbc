# PeanutButterCups

PeanutButterCups (PBC for short) is a feed creation and mirroring
system that runs as web server. Many sources of online content don't
have structured feeds in open syndication formats viz. RSS or Atom,
and if they do, those feeds often only have excerpts of content and
point to a link instead. This makes for a poor experience when you
want to read content via a user agent of your choice, such as an RSS
reader, or want to compile feeds of content into newspaper-like
digests, such as with
[dijester](https://github.com/shrik450/dijester).

At its core, PBC contains of two halves: an ingestion pipeline, which
allows the user to define sources from which data is gathered and
processed; and an Atom output module, which serves Atom feeds from the
ingested data. PBC also comes with an admin interface, via CLI and a
server, to manage it. PBC also has a plugin system, so you can write
code to handle your sources of content or process content in different
ways.

## How it Works

PBC has two core interfaces: _Sources_ and _Views_. Sources are
ingested via the ingestion pipeline to produce _Entries_, which are
then stored in a database. A user can then define a _View_ which picks
up entries and renders them into an Atom feed.

### Sources

PBC's ingestion pipeline has three steps:

1. Ingest
2. Process
3. Write

#### Ingest

PBC takes as input some existing source of content. The Ingest step of
the pipeline gathers content from those sources.

There are two types of ingestion: push and pull. In push ingestion,
the source of the content hits PBC, typically on an HTTP endpoint. In
pull ingestion, PBC itself checks the source on user-defined
intervals. By default, PBC comes with support for RSS and Atom source
feeds for pull ingestion, and a "Bookmark" API source for push
ingestion. Additional source types can be defined by plugins, more on
that later.

#### Process

Once content has been ingested, PBC generates an internal
representation of it, which can then be processed in another
pipeline. For example, the user can choose to filter out some content,
or create a WARC backup of the link associated with the content, or
fetch the link's content and process it via Mozilla's Readability in
cases where the feed only contains excerpts.

There are limitations to the processing pipeline. For one, PBC allows
1 to 1 (mapping) or 1 to 0 (filtering) steps, but doesn't allow n to 1
(reduction) or 1 to n. This means that you can't have a step that
eg. compiles multiple entries into one, or makes multiple entries out
of one.

By default, PBC comes bundled with processing steps to filter based on
title and content; make WARC backups; and fetch and process through
readability. Additional processing steps can be defined with plugins,
again covered later.

#### Write

After entries have been processed, they are written to a database,
usually SQLite.

### Views

A view is a bunch of entries picked via a set of user-defined rules
rendered as an Atom feed. Each view has its own endpoint served by the
pbc server. Views are rendered on-demand, picking a user-defined
number of the most recent entries that much the rules of the
View. However, since PBC persists entries to a database, you can also
see historical versions of a view by including a query parameter.

#### Rules

The rules used to generate a view basically constitute a query over
the entries table. The simplest form of a rule presents all entries in
a single view, this can be useful if you want one canonical feed for
all the content you have. You can also filter by the source the entry
came from or a pattern search on the title or content.
