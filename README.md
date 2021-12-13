# jacygao.github.io

## Usage

This blog site is built with Hugo.

To host the site on localhost, run the following:

```
hugo server -D
```

## Issues

```
found no layout file for "HTML" for kind "page": You should create a template file which matches Hugo Layouts Lookup Rules for this combination.
```

When encoutering the issue above, run the following fix:

```
git submodule init
git submodule update
```

The project should then load without errors.