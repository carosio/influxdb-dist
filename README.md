# influxdb-dist

This repository holds a mirror of the influxdb repository from 
https://github.com/influxdb/influxdb with the difference that all
required dependencies are already present in this repository.
(This is what we also call "vendoring".)

## reasoning

`go get -d` fetches a moving target which is very bad for reproducable
building. For our build-efforts in CAROS we use this mirror instead
of running `go get`. This allows a build-process without prior 
download stage as well as caching the required downloading.

## approach

The approach for creating the files in this repository is to fetch
the desired version from influxdb repository (as tar-ball or via git)
and put it in a directory `$top/src/github.com/influxdb` then with
`GOPATH` set to `$top` a `got get` is used like this:

~~~
go get -d -v github.com/influxdb/influxdb/cmd/influx
go get -d -v github.com/influxdb/influxdb/cmd/influxd
go get -d -v github.com/influxdb/influxdb/cmd/influx_stress
go get -d -v github.com/influxdb/influxdb/cmd/influx_inspect
~~~

Now all required dependencies (for building the targets above) are present
including their `.git` directories for repository meta-data which we
actually do not need. But we might need to know the exact versions later,
so we remove the `.git` repositories but keep the information about
remotes and revisions:

~~~
# (send patches for more elegant ways of doing this, please!)
for x in **/.git
do
  ( 
  cd $( dirname $x )
  git remote -v > .gitreference
  ( git log -n1 |head -1 ) >> .gitreference 
  rm -rf $x
  )
done
# (yea, this was actually a one-liner, but that is quite unreadable)
~~~

After that building works without further downloads:

~~~
go install -v github.com/influxdb/influxdb/cmd/influx
go install -v github.com/influxdb/influxdb/cmd/influxd
go install -v github.com/influxdb/influxdb/cmd/influx_stress
go install -v github.com/influxdb/influxdb/cmd/influx_inspect
~~~
