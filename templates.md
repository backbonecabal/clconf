# Templates

Templates are written in Go's [`text/template`](http://golang.org/pkg/text/template/).

## Template Functions

* [atoi](#atoi)
* [base](#base)
* [base64Decode](#base64decode)
* [base64Encode](#base64encode)
* [cget](#cget)
* [cgets](#cgets)
* [cgetv](#cgetv)
* [cgetvs](#cgetvs)
* [datetime](#datetime)
* [dir](#dir)
* [escapeOsgi](#escapeosgi)
* [exists](#exists)
* [fqdn](#fqdn)
* [get](#get)
* [getenv](#getenv)
* [getksvs](#getksvs)
* [gets](#gets)
* [getsvs](#getsvs)
* [getv](#getv)
* [getvs](#getvs)
* [join](#join)
* [json](#json)
* [jsonArray](#jsonarray)
* [lookupIP](#lookupip)
* [lookupSRV](#lookupsrv)
* [ls](#ls)
* [lsdir](#lsdir)
* [map](#map)
* [parseBool](#parsebool)
* [regexReplace](#regexreplace)
* [replace](#replace)
* [sort](#sort)
* [split](#split)
* [toLower](#tolower)
* [toUpper](#toupper)

### map

creates a key-value map of string -> interface{}

```Text
{{$endpoint := map "name" "elasticsearch" "private_port" 9200 "public_port" 443}}

name: {{index $endpoint "name"}}
private-port: {{index $endpoint "private_port"}}
public-port: {{index $endpoint "public_port"}}
```

specifically useful if you use a sub-template and you want to pass multiple values to it.

### base

Alias for the [path.Base](https://golang.org/pkg/path/#Base) function.

```Text
{{with get "/key"}}
    key: {{base .Key}}
    value: {{.Value}}
{{end}}
```

### exists

Checks if the key exists. Return false if key is not found.

```Text
{{if exists "/key"}}
    value: {{getv "/key"}}
{{end}}
```

### get

Returns the KVPair where key matches its argument. Returns an error if key is not found.

```Text
{{with get "/key"}}
    key: {{.Key}}
    value: {{.Value}}
{{end}}
```

### cget

Returns the KVPair where key matches its argument and the value has been *encrypted*. Returns an error if key is not found.

```Text
{{with cget "/key"}}
    key: {{.Key}}
    value: {{.Value}}
{{end}}
```

### gets

Returns all KVPair, []KVPair, where key matches its argument. Returns an error if key is not found.

```Text
{{range gets "/*"}}
    key: {{.Key}}
    value: {{.Value}}
{{end}}
```

### cgets

Returns all KVPair, []KVPair, where key matches its argument and the values have been *encrypted*.
Returns an error if key is not found.

```Text
{{range cgets "/*"}}
    key: {{.Key}}
    value: {{.Value}}
{{end}}
```

### getv

Returns the value as a string where key matches its argument or an optional default value.
Returns an error if key is not found and no default value given.

```Text
value: {{getv "/key"}}
```

#### With a default value

```Text
value: {{getv "/key" "default_value"}}
```

### cgetv

Returns the *encrypted* value as a string where key matches its argument. Returns an error if key is not found.

```Text
value: {{cgetv "/key"}}
```

### getvs

Returns all values, []string, where key matches its argument. Returns an error if key is not found.

```Text
{{range getvs "/*"}}
    value: {{.}}
{{end}}
```

### cgetvs

Returns all *encrypted* values, []string, where key matches its argument. Returns an error if key is not found.

```Text
{{range cgetvs "/*"}}
    value: {{.}}
{{end}}
```

### getksvs

Returns all values, []string, where key matches its argument, sorted by key. Optionally specify `int` to sort the values as integers. Returns an error if key is not found.

Preserve the order of list inputs:

```bash
$ clconf --pipe getv --template-string '{{getksvs "/foo/*" "int"}}' <<EOF
foo:
- dog
- bird
- cat
EOF
[dog bird cat]
```

Sort by string keys:

```
$ clconf --pipe getv --template-string '{{getksvs "/foo/*"}}' <<EOF
foo:
  dog: woof
  bird: tweet
  cat: meow
EOF
[tweet meow woof]
```

### getsvs

Returns all values, []string, where key matches its argument, sorted. Optionally specify `int` to sort the values as integers. Returns an error if key is not found.

```bash
clconf --pipe getv --template-string '{{getsvs "/foo/*"}}' <<EOF
foo:
- dog
- bird
- cat
EOF
[bird cat dog]
```

### getenv

Wrapper for [os.Getenv](https://golang.org/pkg/os/#Getenv). Retrieves the value of the environment variable named by the key. It returns the value, which will be empty if the variable is not present. Optionally, you can give a default value that will be returned if the key is not present.

```Text
export HOSTNAME=`hostname`
```

```Text
hostname: {{getenv "HOSTNAME"}}
```

#### getenv with a default value

```Text
ipaddr: {{getenv "HOST_IP" "127.0.0.1"}}
```

### datetime

Alias for [time.Now](https://golang.org/pkg/time/#Now)

```Text
# Generated by confd {{datetime}}
```

Outputs:

```Text
# Generated by confd 2015-01-23 13:34:56.093250283 -0800 PST
```

```Text
# Generated by confd {{datetime.Format "Jan 2, 2006 at 3:04pm (MST)"}}
```

Outputs:

```Text
# Generated by confd Jan 23, 2015 at 1:34pm (EST)
```

See the time package for more usage: [http://golang.org/pkg/time/](http://golang.org/pkg/time/)

### split

Wrapper for [strings.Split](http://golang.org/pkg/strings/#Split). Splits the input string on the separating string and returns a slice of substrings.

```Text
{{ $url := split (getv "/deis/service") ":" }}
    host: {{index $url 0}}
    port: {{index $url 1}}
```

### toUpper

Alias for [strings.ToUpper](http://golang.org/pkg/strings/#ToUpper) Returns uppercased string.

```Text
key: {{toUpper "value"}}
```

### toLower

Alias for [strings.ToLower](http://golang.org/pkg/strings/#ToLower). Returns lowercased string.

```Text
key: {{toLower "Value"}}
```

### json

Returns an map[string]interface{} of the json value.

### lookupSRV

Wrapper for [net.LookupSRV](https://golang.org/pkg/net/#LookupSRV). The wrapper also sorts the SRV records alphabetically by combining all the fields of the net.SRV struct to reduce unnecessary config reloads.

```Text
{{range lookupSRV "mail" "tcp" "example.com"}}
  target: {{.Target}}
  port: {{.Port}}
  priority: {{.Priority}}
  weight: {{.Weight}}
{{end}}
```

### base64Encode

Returns a base64 encoded string of the value.

```Text
key: {{base64Encode "Value"}}
```

### base64Decode

Returns the string representing the decoded base64 value.

```Text
key: {{base64Decode "VmFsdWU="}}
```

#### Add keys to etcd

```Text
etcdctl set /services/zookeeper/host1 '{"Id":"host1", "IP":"192.168.10.11"}'
etcdctl set /services/zookeeper/host2 '{"Id":"host2", "IP":"192.168.10.12"}'
```

#### Create the template resource

```toml
[template]
src = "services.conf.tmpl"
dest = "/tmp/services.conf"
keys = [
  "/services/zookeeper/"
]
```

#### Create the template

```Text
{{range gets "/services/zookeeper/*"}}
{{$data := json .Value}}
  id: {{$data.Id}}
  ip: {{$data.IP}}
{{end}}
```

#### Advanced Map Traversals

Once you have parsed the JSON, it is possible to traverse it with normal Go
template functions such as `index`.

A more advanced structure, like this:

```json
{
  "animals": [
    {"type": "dog", "name": "Fido"},
    {"type": "cat", "name": "Misse"}
  ]
}
```

It can be traversed like this:

```Text
{{$data := json (getv "/test/data/")}}
type: {{ (index $data.animals 1).type }}
name: {{ (index $data.animals 1).name }}
{{range $data.animals}}
{{.name}}
{{end}}
```

### jsonArray

Returns a []interface{} from a json array such as `["a", "b", "c"]`.

```Text
{{range jsonArray (getv "/services/data/")}}
val: {{.}}
{{end}}
```

### ls

Returns all subkeys, []string, where path matches its argument. Returns an empty list if path is not found.

```Text
{{range ls "/deis/services"}}
   value: {{.}}
{{end}}
```

### lsdir

Returns all subkeys, []string, where path matches its argument. It only returns subkeys that also have subkeys. Returns an empty list if path is not found.

```Text
{{range lsdir "/deis/services"}}
   value: {{.}}
{{end}}
```

### dir

Returns the parent directory of a given key.

```Text
{{with dir "/services/data/url"}}
dir: {{.}}
{{end}}
```

### join

Alias for the [strings.Join](https://golang.org/pkg/strings/#Join) function.

```Text
{{$services := getvs "/services/elasticsearch/*"}}
services: {{join $services ","}}
```

### replace

Alias for the [strings.Replace](https://golang.org/pkg/strings/#Replace) function.

```Text
{{$backend := getv "/services/backend/nginx"}}
backend = {{replace $backend "-" "_" -1}}
```

### lookupIP

Wrapper for [net.LookupIP](https://golang.org/pkg/net/#LookupIP) function. The wrapper also sorts (alphabeticaly) the IP addresses. This is crucial since in dynamic environments DNS servers typically shuffle the addresses linked to domain name. And that would cause unnecessary config reloads.

```Text
{{range lookupIP "some.host.local"}}
    server {{.}};
{{end}}
```

### atoi

Alias for the [strconv.Atoi](https://golang.org/pkg/strconv/#Atoi) function.

```Text
{{seq 1 (atoi (getv "/count"))}}
```

### parseBool

An alias to [`strconv.ParseBool`](https://golang.org/pkg/strconv/#ParseBool)

### regexReplace

Given a regex, an original string and a replacement string run [`regexp.ReplaceAllString`](https://golang.org/pkg/regexp/#Regexp.ReplaceAllString) and return the result. Returns an error if the regex fails to compile.

```bash
$ clconf --pipe getv --template-string '{{regexReplace "o+" "foo" "e"}}' < /dev/null
fe
```

### escapeOsgi

Places a single `\` prior to any `'`, `"`, `\`, `=` or space.

```bash
$ clconf --pipe getv --template-string '{{escapeOsgi "foo=bar"}}' < /dev/null
foo\=bar
```

### fqdn

Adds a domain to a hostname if not already qualified.

```bash
$ clconf --pipe getv --template-string '{{fqdn "foo" "example.com"}}' < /dev/null
foo.example.com
$ clconf --pipe getv --template-string '{{fqdn "foo.google.com" "example.com"}}' < /dev/null
foo.google.com
```

### sort

Sorts the input ([]interface{}) by translating it to the specified type (one of `int`, `string`, default: `string`)

```bash
clconf --pipe getv --template-string '{{sort (getvs "/foo/*") }}' <<EOF
foo:
- dog
- bird
- cat
EOF
[bird cat dog]
```

## Example Usage

Given the yaml input:

```yaml
---
nginx:
  domain: example.com
  root: /var/www/example_dotcom
  worker_processes: 2
app:
  upstream:
    app1: 10.0.1.100:80
    app2: 10.0.1.101:80
```

And the template:

```Text
worker_processes {{getv "/nginx/worker_processes"}};

upstream app {
{{range getvs "/app/upstream/*"}}
    server {{.}};
{{end}}
}

server {
    listen 80;
    server_name www.{{getv "/nginx/domain"}};
    access_log /var/log/nginx/{{getv "/nginx/domain"}}.access.log;
    error_log /var/log/nginx/{{getv "/nginx/domain"}}.log;

    location / {
        root              {{getv "/nginx/root"}};
        index             index.html index.htm;
        proxy_pass        http://app;
        proxy_redirect    off;
        proxy_set_header  Host             $host;
        proxy_set_header  X-Real-IP        $remote_addr;
        proxy_set_header  X-Forwarded-For  $proxy_add_x_forwarded_for;
    }
}
```

Output:

```Text
worker_processes 2;

upstream app {
    server 10.0.1.100:80;
    server 10.0.1.101:80;
}

server {
    listen 80;
    server_name www.example.com;
    access_log /var/log/nginx/example.com.access.log;
    error_log /var/log/nginx/example.com.error.log;

    location / {
        root              /var/www/example_dotcom;
        index             index.html index.htm;
        proxy_pass        http://app;
        proxy_redirect    off;
        proxy_set_header  Host             $host;
        proxy_set_header  X-Real-IP        $remote_addr;
        proxy_set_header  X-Forwarded-For  $proxy_add_x_forwarded_for;
    }
}
```

### Complex example

This examples show how to use a combination of the templates functions to do nested iteration.

```Text
{{range $dir := lsdir "/services/web"}}
upstream {{base $dir}} {
    {{$custdir := printf "/services/web/%s/*" $dir}}{{range gets $custdir}}
    server {{$data := json .Value}}{{$data.IP}}:80;
    {{end}}
}

server {
    server_name {{base $dir}}.example.com;
    location / {
        proxy_pass {{base $dir}};
    }
}
{{end}}
```

Output:

```Text
upstream cust1 {
    server 10.0.0.1:80;
    server 10.0.0.2:80;
}

server {
    server_name cust1.example.com;
    location / {
        proxy_pass cust1;
    }
}

upstream cust2 {
    server 10.0.0.3:80;
    server 10.0.0.4:80;
}

server {
    server_name cust2.example.com;
    location / {
        proxy_pass cust2;
    }
}
```

Go's [`text/template`](http://golang.org/pkg/text/template/) package is very powerful. For more details on it's capabilities see its [documentation.](http://golang.org/pkg/text/template/)
