# flumeview-reduce

A flumeview into a reduce function.
Stream append-only log data into a reduce function to calculate a state.

## Example

``` js
var FlumeLog = require('flumelog-offset')
var codec = require('flumecodec')
var Flume = require('flumedb')
var Reduce = require('flumeview-reduce')

//statistics exports a reduce function that calculates
//mean, stdev, etc!
var statistics = require('statistics')

//initialize a flumelog with a codec.
//this example uses flumelog-offset, but any flumelog is valid.
var log = FlumeLog(file, 1024*16, codec.json) //use any flume log

//attach the reduce function.
var db = Flume(log).use('stats',
    Reduce(1, statistics, function (data) {
      return data.value
    })

db.append({value: 1}, function (err) {

  db.stats.get(function (err, stats) {
    console.log(stats) // => {mean: 1, stdev: 0, count: 1, sum: 1, ...}
  })
})
```

## FlumeViewReduce(version, reduce, map?) => FlumeView

construct a flumeview from this reduce function. `version` should be a number,
and must be provided. If you make a breaking change to either `reduce` or `map`
then increment `version` and the view will be rebuilt.

`map` is optional. If map is applied, then each item in the log is passed to `map`
and then if the returned value is not null, it is passed to reduce.

``` js
var _data = map(data)
if(_data != null)
  state = reduce(state, map(data))
```

using a `map` function is useful, because it enables efficiently streaming the realtime
changes in the state to a remote client.

then, pass the flumeview to `db.use(name, flumeview)`
and you'll have access to the flumeview methods on `db[name]...`

## db[name].get(cb)

get the current state of the reduce. This will wait until the view is up to date, if necessary.

## db[name].stream({live: boolean}) => PullSource

Stream the changing reduce state. for this to work, a map function must be provided.

If so, the same reduce function can be used to process the output.

## License

MIT

