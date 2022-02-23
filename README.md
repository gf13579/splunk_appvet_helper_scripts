# Splunk App Vetting Helper Scripts

A bunch of one-liners and script to help with bulk app vetting during cloud migrations

## Separate apps with local content

The following command identifies apps with `local/[something]` content and moves them all into a new folder (_splunkbase) for app inspection:

```bash
mkdir _splunkbase; find . -mindepth 2 -maxdepth 3 | egrep -e "local/.+" | cut -d "/" -f 2 | sort | uniq | xargs -I {} mv '{}' _splunkbase
```

Then maybe check the remainder for any interesting lookups:

```bash
find . | egrep "lookups/.*\.csv"
```

## Bulk update of dashboard versions

Blindly assumes all dashes are compatible with the jquery upgrade in Splunk 8.something. Deliberately won't add a version attribute if one appears to be present.

### Preview

```bash
find . -wholename "*local/data/ui/views/*.xml" -exec egrep -i -e '<(dashboard|form)(\s|>)' -l {} \; | xargs -I {} egrep -i '<(dashboard|form)\s[^>]*version' -L {} | xargs -I {} sed -r 's/(<(dashboard|form)[^>]*)>/\1 version="1.1">/g' {}| egrep "<(dashboard|form)"
```

### Make changes

```bash
find . -wholename "*local/data/ui/views/*.xml" -exec egrep -i -e '<(dashboard|form)(\s|>)' -l {} \; | xargs -I {} egrep -i '<(dashboard|form)\s[^>]*version' -L {} | xargs -I {} sed -i -r 's/(<(dashboard|form)[^>]*)>/\1 version="1.1">/g' {}
```

## Bulk update of local.meta

### Preview

```bash
find . -wholename "*metadata/local.meta" -exec grep -i -P '[\s,=]sc_admin[\s,^]' -L {} \; | xargs -I {} sed -r -e 's/owner\s*+*=\s*admin/owner = sc_admin/g' -e 's/(\[[^]]*\sadmin)\s([^]]*\])/\1, sc_admin \2/g' {} | grep sc_admin
```

### Make changes

```bash
find . -wholename "*metadata/local.meta" -exec grep -i -P '[\s,=]sc_admin[\s,^]' -L {} \; | xargs -I {} sed -i -r -e 's/owner\s*=\s*admin/owner = sc_admin/g' -e 's/(\[[^]]*\sadmin)\s([^]]*\])/\1, sc_admin \2/g' {}
```

