ubergarm.com
===
My personal website lovingly crafted in `docute`.

## Build
```bash
docker build -t ubergarm/ubergarm.com .
```

## Run
```bash
docker run --rm -it -v `pwd`:/app -w /app -p 8080:8080 -p 35729:35729 ubergarm/ubergarm.com /bin/sh
```

## Dev
```bash
yarn global add docute-cli
docute init ./docs
docute --watch ./docs
```

## References
* [egoist/docute](https://github.com/egoist/docute)
