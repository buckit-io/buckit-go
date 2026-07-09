For maintainers only
====================

### Making new releases

Tag and sign your release commit, additionally this step requires you to have access to MinIO's trusted private key.

```sh
$ git tag -s 4.0.0
$ git push
$ git push --tags
```

### Update version

Once release has been made update `libraryVersion` constant in `api.go` to next to be released version.

```sh
$ grep libraryVersion api.go
      libraryVersion = "4.0.1"
```

Commit your changes

```
$ git commit -a -m "Update version for next release" --author "<your name>"
```

### Announce

Release notes requires two sections `highlights` and `changelog`. Highlights is a bulleted list of salient features in this release and Changelog contains list of all commits since the last release.

To generate `changelog`

```sh
$ git log --no-color --pretty=format:'-%d %s (%cr) <%an>' <last_release_tag>..<latest_release_tag>
```
