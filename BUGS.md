# Bugs

<strike>## Green does not work

No idea what's wrong with it. 

```
$ curl -s -H "Host: canary.default.example.com" http://35.228.80.221
<!DOCTYPE html>
<html lang="en">
<head>
    <title>Knative Routing Demo</title>
    <link rel="stylesheet" type="text/css" href="/css/app.css" />
</head>
<body>
        
            <div class="blue">App v1</div>
        
    </div>
</body>
</html>asko@Castle knative-lab (master) $ curl -s -H "Host: canary.default.example.com" http://35.228.80.221
```

i.e. then in the blue/green deployment a green end point would be hit, it does not respond.
</strike>

Got it running. Not sure how, yet. Maybe the adding of node pool to 1..10 (was 3..3) helped. Maybe something else I changed.