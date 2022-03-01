# Regex Magic

[[toc]]

## Select from start to stop ignoring \n etc

```regex
START[\w\W]*?STOP
```

Every time I have needed to use it it worked perfectly and then I realized there was a better way to do what I was doing and i'm dumb but it could be useful I think

## Everything but

```regex
[^\[\], abcd]
```

not `[,],,, ,a,b,c,d`
