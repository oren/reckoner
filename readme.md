# Reckoner
This is a continuation of my personal work from [here](http://blog.thefrontiergroup.com.au/2015/05/blockchain-analytics-with-cayley-db/). This project will parse the entire blockchain into a [Cayley](cayley.io), a graph database.

## Installation

### Dependencies

- btcd
- dgraph
- Go 1.6 or above

## Setup

### btcd

```
git clone https://github.com/btcsuite/btcd $GOPATH/src/github.com/btcsuite/btcd
cd $GOPATH/src/github.com/btcsuite/btcd
git pull && glide install
go install . ./cmd/...
btcd --txindex --rpcuser=user --rpcpass=pass
```

### reckoner

```
glide install
go build
./reckoner parse > triples
wc -l triples
tail triples
gzip triples
cayley init -db bolt
cayley load --quads ./triples.gz -db bolt -format nquad -ignoremissing -ignoredup
cayley http -db bolt
```

Open the browser at [localhost:64210]()

## Sample Queries

![visualization](http://imgur.com/iU5E9tw)
```
g.V("<block.1>").Tag("source")
.Out("<bitcoin.height.block>").Tag("target")
.Out("<bitcoin.block.nextblock>")
.Out("<bitcoin.block.nextblock>")
.Out("<bitcoin.block.nextblock>")
.Out("<bitcoin.block.nextblock>")
.Out("<bitcoin.block.nextblock>")
.Tag("target").All()
```

![visualization](http://puu.sh/pUdhq/db31ac029b.png)
```
firstSource = g.V("<block.1>").Tag("source").Out("<bitcoin.height.block>")
var newSource;
var currSource = firstSource;
var pairs = [];

for (var i=0; i<10; i++) {
  newSource = currSource.Out("<bitcoin.block.nextblock>").Tag("target");
	if (!newSource) break;
	pairs.push({
		source: currSource.ToArray(),
		target: newSource.ToArray()
	});

	currSource = newSource;
}

for (var i=0; i<pairs.length; i++) {
	g.Emit({source: pairs[i].source[0], target: pairs[i].target[0]});
}
```

## Contributing

1. Fork it!
2. Create your feature branch: `git checkout -b my-new-feature`
3. Commit your changes: `git commit -am 'Add some feature'`
4. Push to the branch: `git push origin my-new-feature`
5. Submit a pull request :D
