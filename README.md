# pprofsnapshot

pprofsnapshot makes it easy to download profiling data to local.

## Usages

pprofsnapshot packages profiles as *.tar.gz archived, it offers the std http.Handle which can be embedded in your code easily.

```golang
package main

import (
	"net/http"

	"github.com/chenjiandongx/pprofsnapshot"
)

func main() {
	// http handler provides some query params similar to the `http/pprof`
	// * seconds
	// * debug
	// * profiles: goroutine/threadcreate/heap/allocs/block/mutex/cpu, split by comma
	http.Handle("/debug/pprof/snapshot", pprofsnapshot.HandlerFor())
    // 
	// http.HandleFunc("/debug/pprof/snapshot", pprofsnapshot.HandlerFuncFor())
	http.ListenAndServe(":8080", nil)
}

/*
for example:

> $ curl http://localhost:8080/debug/pprof/snapshot?debug=1&seconds=15&profiles=heap,cpu,goroutine -o profiles.tar.gz
% Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
Dload  Upload   Total   Spent    Left  Speed
100  3143    0  3143    0     0    209      0 --:--:--  0:00:15 --:--:--   658

> $ tar -zxvf profiles.tar.gz
x profiles/heap.pprof
x profiles/cpu.pprof
x profiles/goroutine.pprof
*/
```

Or just leverage the `Collector` to capture profiling data anywhere like this.

```golang
// 1) Collect API
collector := pprofsnapshot.NewCollector()
b, err := collector.Collect(context.Background())
if err != nil {
	// handles error
}
f, err := os.Create("profiles.tar.gz")
if err != nil {
	// handles error
}
f.Write(b)

// 2) Write API
collector := pprofsnapshot.NewCollector()
f, err := os.Create("profiles.tar.gz")
if err != nil {
	// handles error
}
collector.Write(context.Background(), f)
```

### Options

```golang
// WithDebugLevel sets profiles debug level
WithDebugLevel(level int) Option

// WithEnabledProfiles sets collected profiles.
// supports goroutine/threadcreate/heap/allocs/block/mutex/cpu
WithEnabledProfiles(profiles []string) Option

// WithSamplingSeconds sets sampling duration in seconds.
WithSamplingSeconds(n int) Option
```

### License

MIT [©chenjiandongx](https://github.com/chenjiandongx)
