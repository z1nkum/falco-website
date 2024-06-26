---
exclude_search: true
title: Falco അലേർട്ട്സ്
weight: 8
---

Falco ക്ക് ഒന്നോ അതിൽ കൂടുതലോ ചാനലുകളിലേക്ക് അലേർട്ട്സ് അയക്കാൻ സാധിക്കും.

* സ്റ്റാൻഡേർഡ് ഔട്ട്പുട്ട്
* ഒരു ഫയൽ
* Syslog
* ഒരു വികസിപ്പിച്ച പ്രോഗ്രാം
* ഒരു HTTP[s] എൻഡ് പോയിൻറ്
* gRPC API ഉപയോഗിക്കുന്ന ക്ലൈന്റ്‌

`falco.yaml` എന്ന ഫാൽക്കോ ക്രമീകരണ ഫയൽ വഴി ചാനലുകൾ ക്രമീകരിച്ചിരിക്കുന്നു. കൂടുതൽ വിവരങ്ങൾക്കായി [Falco Configuration](../configuration) എന്ന പേജ് കാണുക. ആ ചാനലുകൾ ഓരോന്നിനെയും കുറിച്ചുള്ള വിവരങ്ങൾ ഇതാ.

## സ്റ്റാൻഡേർഡ് ഔട്ട്പുട്ട്

സ്റ്റാൻഡേർഡ് ഔട്ട്പുട്ട് വഴി അലേർട്ട്സ് അയക്കാനായി ക്രമീകരിക്കപ്പെടുമ്പോൾ, ഓരോ അലേർട്ടിനുമായി ഒരു വരി പ്രിൻറ് ചെയ്യപ്പെടും. ഇതാ ഒരു ഉദാഹരണം:

```yaml
stdout_output:
  enabled: true
```

```
10:20:05.408091526: Warning Sensitive file opened for reading by non-trusted program (user=root command=cat /etc/shadow file=/etc/shadow)
```
കണ്ടെയ്നേർസിൽ നിന്നും ലോഗ്സ് കാപ്ച്ചർ ചെയ്യുന്നതിന് Fluentd അല്ലെങ്കിൽ Logstash ഉപയോഗിക്കുമ്പോൾ സ്റ്റാൻഡേർഡ് ഔട്ട്പുട്ട് ഉപയോഗപ്രദമാണ്. അങ്ങനെ അലേർട്ട്സ് Elasticsearch ൽ ശേഖരിക്കാൻ കഴിയും, കൂടാതെ അലേർട്ട്സ് ദൃശ്യവൽക്കരിക്കാൻ ഡാഷ്ബോർഡ്സ് നിർമ്മിക്കാവുന്നതുമാണ്. കൂടുതൽ വിവരങ്ങൾക്കായി [this blog post](https://sysdig.com/blog/kubernetes-security-logging-fluentd-falco/) വായിക്കുക.

`-d/--daemon` കമാൻഡ് ലൈൻ ഓപ്ഷൻ വഴി പശ്ചാത്തലത്തിൽ റൺ ചെയ്യുമ്പോൾ, സ്റ്റാൻഡേർഡ് ഔട്ട്പുട്ട് സന്ദേശങ്ങൾ നിരാകരിക്കപ്പെടും.

## ഫയൽ ഔട്ട്പുട്ട്
ഒരു ഫയലിലേക്ക് അലേർട്ട്സ് അയക്കാനായി ക്രമീകരിക്കപ്പെടുമ്പോൾ, ഒരു സന്ദേശം ഓരോ അലേർട്ടിനും വേണ്ടി ഫയലിലേക്ക് എഴുതപ്പെടുന്നു. രൂപകൽപ്പന ഈ സ്റ്റാൻഡേർഡ് ഔട്ട്പുട്ട് രൂപകൽപ്പനയോട് വളരെ സദൃശമാണ്:

```yaml
file_output:
  enabled: true
  keep_alive: false
  filename: ./events.txt
```

`keep_alive` തെറ്റാകുമ്പോൾ(ഡീഫോൾട്ട്), ഓരോ അലേർട്ടിനും വേണ്ടി  കൂട്ടിച്ചേർക്കലിനായി ഫയൽ ഓപ്പൺ ചെയ്യപ്പെടുകയും, ഒരു സിംഗിൾ അലേർട്ട് എഴുതപ്പെടുകയും, ഫയൽ ക്ലോസ് ചെയ്യപ്പെടുകയും ചെയ്യുന്നു. ഫയൽ റൊട്ടേറ്റ് ചെയ്തതോ അല്ലെങ്കിൽ വെട്ടിച്ചുരുക്കിയതോ അല്ല. `keep_alive` ശരിയായിട്ടാണ് സജ്ജീകരിക്കുന്നതെങ്കിൽ ആദ്യ അലേർട്ടിന് മുൻപ് ഫയൽ ഓപ്പൺ ചെയ്യപ്പെടുകയും തുടർന്നുള്ള എല്ലാ അലേർട്ട്സിനായും തുറന്നുവക്കുകയും ചെയ്യുന്നു. ഔട്ട്പുട്ട് ബഫർ ചെയ്യുകയും ക്ലോസ് ചെയ്യുമ്പോൾ മാത്രം ഫ്ലഷ് ചെയ്യുകയും ചെയ്യും. ( ഇത് `--unbuffered` ഉപയോഗിച്ച് മാറ്റാവുന്നതാണ്. )

ഔട്ട്പുട്ട് ഫയൽ റൊട്ടേറ്റ് ചെയ്യുന്നതിന് [logrotate](https://github.com/logrotate/logrotate) പോലെയുള്ള ഒരു പ്രോഗ്രാം ഉപയോഗിക്കണമെന്നുണ്ടെങ്കിൽ, ഒരു ലോഗോറൊട്ടേറ്റ് കോൺഫിഗ് ഉദാഹരണം [ഇവിടെ](https://github.com/falcosecurity/falco/blob/ffd8747ec0943db2546c3270826e1700dc4df75f/examples/logrotate/falco). ലഭ്യമാണ്.
Falco 0.10.0 പ്രകാരം, `SIGUSR1` ഉപയോഗിച്ച് സിഗ്നൽ ചെയ്യപ്പെടുമ്പോൾ ഫാൽക്കോ അതിൻറെ ഫയൽ ഔട്ട്പുട്ട് ക്ലോസ് ചെയ്യുകയും റീഓപ്പൺ ചെയ്യുകയും ചെയ്യും. മുകളിൽ കൊടുത്തിട്ടുള്ള ലോഗ്റൊട്ടേറ്റ് ഉദാഹരണം അതിനെ ആശ്രയിച്ചിരിക്കുന്നു.

## സിസ്ലോഗ് ഔട്ട്പുട്ട്
സിസ്ലോഗിലേക്ക് അലേർട്ട്സ് അയക്കാൻ ക്രമീകരിക്കപ്പെടുമ്പോൾ, ഓരോ അലേർട്ടിനും ഒരു സിസ്ലോഗ് സന്ദേശം അയക്കപ്പെടുന്നു. യഥാർത്ഥ രൂപകൽപ്പന നിങ്ങളുടെ സിസ്ലോഗ് ഡെയ്മണിനെ ആശ്രയിച്ചിരിക്കുന്നു, പക്ഷേ ഇതാ ഒരു ഉദാഹരണം:

```yaml
syslog_output:
  enabled: true
```

```
Jun  7 10:20:05 ubuntu falco: Sensitive file opened for reading by non-trusted program (user=root command=cat /etc/shadow file=/etc/shadow)
```
`LOG_USER` സൌകര്യമുപയോഗിച്ചാണ് സിസ്ലോഗ് സന്ദേശങ്ങൾ അയക്കപ്പെടുന്നത്. നിയമത്തിൻറെ മുൻഗണനയാണ് സിസ്ലോഗ് സന്ദേശത്തിൻറെ മുൻഗണനയായി ഉപയോഗിക്കുന്നത്.

## പ്രോഗ്രാം ഔട്ട്പുട്ട്
ഒരു പ്രോഗ്രാമിലേക്ക് അലേർട്ട്സ് അയക്കാൻ ക്രമീകരിക്കപ്പെടുമ്പോൾ, ഫാൽക്കോ ഓരോ അലേർട്ടിനുമായി പ്രോഗ്രാം ആരംഭിക്കുകയും, അതിൻറെ ഉള്ളടക്കങ്ങൾ പ്രോഗ്രാമിൻറെ സ്റ്റാൻഡേർഡ് ഇൻപുട്ടിലേക്ക് എഴുതുകയും ചെയ്യുന്നു. നിങ്ങൾക്ക് ഒരേസമയം ഒരു സിംഗിൾ പ്രോഗ്രാം ഔട്ട്പുട്ട് മാത്രമേ ക്രമീകരിക്കാൻ സാധിക്കൂ. ഉദാ: ഒരു സിംഗിൾ പ്രോഗ്രാമിലേക്കുള്ള റൂട്ട് അലേർട്ട്സ്.

ഉദാഹരണത്തിന് ഒന്നിൻറെ `falco.yaml` ക്രമീകരണം നൽകിയിരിക്കുന്നു.

```yaml
program_output:
  enabled: true
  keep_alive: false
  program: mail -s "Falco Notification" someone@example.com
```
പ്രോഗ്രാമിന് സാധാരണയായി സ്റ്റാൻഡേർഡ് ഇൻപുട്ടിൽ നിന്നും ഇൻപുട്ട് സ്വീകരിക്കാൻ കഴിയുന്നില്ലെങ്കിൽ, ഒരു ആർഗ്യുമെൻറ് ഉപയോഗിച്ച് ഫാൽക്കോ ഇവൻറുകൾ കൈമാറാൻ `xargs` ഉപയോഗിക്കാവുന്നതാണ്.
ഉദാഹരണത്തിന്:

```yaml
program_output:
  enabled: true
  keep_alive: false
  program: "xargs -I {} aws --region ${region} sns publish --topic-arn ${falco_sns_arn} --message {}"
```

`keep_alive` തെറ്റാകുമ്പോൾ (ഡീഫോൾട്ട് ), ഓരോ അലേർട്ടിനും വേണ്ടി ഫാൽക്കോ `mail -s …` എന്ന പ്രോഗ്രാം റൺ ചെയ്യുകയും, അലേർട്ട് പ്രോഗ്രാമിലേക്ക് എഴുതുകയും ചെയ്യും. പ്രോഗ്രാം ഒരു ഷെൽ വഴിയാണ് റൺ ചെയ്യുന്നത്, അതിനാൽ നിങ്ങൾ അധിക രൂപകൽപ്പന ചെയ്യാൻ ആഗ്രഹിക്കുന്നുവെങ്കിൽ,ഒരു കമാൻഡ് പൈപ്പ്ലൈൻ വ്യക്തമാക്കുന്നത് സാധ്യമാണ്.

`keep_alive` ശരിയായിട്ടാണ് സജ്ജീകരിക്കുന്നതെങ്കിൽ, ആദ്യ അലേർട്ടിന് മുൻപ് ഫാൽക്കോ പ്രോഗ്രാം വികസിപ്പിക്കുകയും അലേർട്ട് എഴുതുകയും ചെയ്യും. തുടർന്നുള്ള എല്ലാ അലേർട്ട്സിനായും പൈപ്പ് തുറന്നുവക്കുന്നു. ഔട്ട്പുട്ട് ബഫർ ചെയ്യപ്പെടുകയും ക്ലോസ് ചെയ്യുമ്പോൾ മാത്രം ഫ്ലഷ് ചെയ്യുകയും ചെയ്യും. ( ഇത് `--unbuffered` ഉപയോഗിച്ച് മാറ്റാവുന്നതാണ്.

കുറിപ്പ്:  ഫാൽക്കോ വികസിപ്പിക്കുന്ന പ്രോഗ്രാം ഫാൽക്കോയുടെ അതേ പ്രോസസ്സ് ഗ്രൂപ്പിലാണ്, കൂടാതെ ഫാൽക്കോക്ക് ലഭിക്കുന്ന എല്ലാ സിഗ്നലുകളും അതിന് ലഭിക്കും.  ബഫർ ചെയ്ത ഔട്ട്പുട്ടുകൾക്ക് മുന്നിൽ ഒരു ക്ലീൻ ഷട്ട്ഡൌൺ അനുവദിക്കാനായി നിങ്ങൾക്ക് SIGTERM അവഗണിക്കണമെന്നുണ്ടെങ്കിൽ,  നിങ്ങൾ സ്വയം തന്നെ സിഗ്നൽ ഹാൻഡ്ലർ അസാധുവാക്കണം.

Falco 0.10.0 പ്രകാരം, `SIGUSR1` ഉപയോഗിച്ച് സിഗ്നൽ ചെയ്യപ്പെടുമ്പോൾ ഫാൽക്കോ അതിൻറെ ഫയൽ ഔട്ട്പുട്ട് ക്ലോസ് ചെയ്യുകയും റീഓപ്പൺ ചെയ്യുകയും ചെയ്യും.

## പ്രോഗ്രാം ഔട്ട്പുട്ട് ഉദാഹരണം: സ്ലാക്ക് ഇൻകമിങ് വെബ്ഹുക്കിലേക്ക് പോസ്റ്റ് ചെയ്യുന്നത്

നിങ്ങൾക്ക് ഒരു സ്ലാക്ക് ചാനലിലേക്ക് ഫാൽക്കോ അറിയിപ്പുകൾ അയക്കണമെന്നുണ്ടെങ്കിൽ, സ്ലാക്ക് വെബ്ഹുക്ക് അന്തിമപോയിൻറിന് ആവശ്യമായ രൂപത്തിലേക്ക് JSON ഔട്ട്പുട്ട് മസ്സാജ് ചെയ്യുന്നതിന് ആവശ്യമായ ക്രമീകരണം ഇതാ:

```yaml
# Whether to output events in json or text
json_output: true
…
program_output:
  enabled: true
  program: "jq '{text: .output}' | curl -d @- -X POST https://hooks.slack.com/services/XXX"
```
### പ്രോഗ്രാം ഔട്ട്പുട്ട്: നെറ്റ്വർക്ക് ചാനലിലേക്ക് അലേർട്ട്സ് അയക്കുന്നത്

നിങ്ങൾക്ക് ഒരു നെറ്റ്വർക്ക് കണക്ഷനിലൂടെ അലേർട്ട്സിൻറെ ഒരു സ്ട്രീം അയക്കണമെന്നുണ്ടെങ്കിൽ, ഇതാ ഒരു ഉദാഹരണം:

```yaml
# Whether to output events in json or text
json_output: true
…
program_output:
  enabled: true
  keep_alive: true
  program: "nc host.example.com 1234"
```
നെറ്റ്വർക്ക് കണക്ഷൻ സ്ഥായിയായി നിലനിർത്തുന്നതിന് `keep_alive: true` ൻറെ ഉപയോഗം ശ്രദ്ധിക്കുക.

## HTTP[s] ഔട്ട്പുട്ട്: ഒരു HTTP[s] അന്തിമ പോയിൻറിലേക്ക് അലേർട്ട്സ് അയക്കുക
നിങ്ങൾക്ക് ഒരു HTTP[s] അന്തിമ പോയിൻറിലേക്ക് അലേർട്ട്സ് അയക്കണമെന്നുണ്ടെങ്കിൽ, `http_output` എന്ന ഓപ്ഷൻ ഉപയോഗിക്കാവുന്നതാണ്:

```yaml
json_output: true
...
http_output:
  enabled: true
  url: http://some.url/some/path/
```

നിലവിൽ എൻക്രിപ്റ്റ് ചെയ്യപ്പെടാത്ത HTTP അന്തിമ പോയിൻറുകൾ അല്ലെങ്കിൽ സാധുവായതും, സുരക്ഷിതമായതുമായ HTTPs അന്തിമപോയിൻറുകൾ മാത്രമേ പിന്തുണക്കൂ.( അതായത്, അസാധുവോ സ്വയം ഒപ്പുവെച്ചതോ ആയ സാക്ഷ്യപത്രങ്ങൾ പിന്തുണക്കുന്നതല്ല.)

## JSON ഔട്ട്പുട്ട്
എല്ലാ ഔട്ടപുട്ട് ചാനലുകൾക്കായും, നിങ്ങൾക്ക് ക്രമീകരണ ഫയലിലോ അല്ലെങ്കിൽ കമാൻഡ് ലൈനിലോ JSON ഔട്ട്പുട്ടിലേക്ക് മാറാവുന്നതാണ്. ഓരോ അലേർട്ടിനായും ഫാൽക്കോ, ഇനി പറയുന്ന സവിശേഷതകൾ അടങ്ങിയ ഒരു സിംഗിൾ ലൈനിലുള്ള ഒരു JSON വസ്തു പ്രിൻറ് ചെയ്യും:

* time: ISO8601 രൂപകൽപ്പനയിലുള്ള, അലേർട്ടിൻറെ സമയം.
* rule : അലേർട്ടിന് കാരണമായ നിയമം.
* priority : അലേർട്ട് സൃഷ്ടിച്ച നിയമത്തിൻറെ മുൻഗണന.
* output: അലേർട്ടിനായി രൂപകൽപ്പന ചെയ്ത ഔട്ട്പുട്ട് സ്ട്രിങ്.
* output_fields : ഔട്ട്പുട്ട് എക്സ്പ്രഷനിലെ ഓരോ ടെംപ്ലേറ്റഡ്  മൂല്യത്തിനും, അലേർട്ടിന് പ്രേരിപ്പിച്ച ഇവൻറിൽ നിന്നുമുള്ള ആ ഫീൽഡിൻറെ മൂല്യം.

ഇതാ ഒരു ഉദാഹരണം:

```javascript
{"output":"16:31:56.746609046: Error File below a known binary directory opened for writing (user=root command=touch /bin/hack file=/bin/hack)","priority":"Error","rule":"Write below binary dir","time":"2017-10-09T23:31:56.746609046Z", "output_fields": {"evt.t\
ime":1507591916746609046,"fd.name":"/bin/hack","proc.cmdline":"touch /bin/hack","user.name":"root"}}
```

ഇതാ അതേ ഔട്ട്പുട്ട്, മനോഹരമായി പ്രിൻറ് ചെയ്തത്:

```javascript
{
   "output" : "16:31:56.746609046: Error File below a known binary directory opened for writing (user=root command=touch /bin/hack file=/bin/hack)"
   "priority" : "Error",
   "rule" : "Write below binary dir",
   "time" : "2017-10-09T23:31:56.746609046Z",
   "output_fields" : {
      "user.name" : "root",
      "evt.time" : 1507591916746609046,
      "fd.name" : "/bin/hack",
      "proc.cmdline" : "touch /bin/hack"
   }
}
```
## gRPC ഔട്ട്പുട്ട്

നിങ്ങൾക്ക് gRPC API വഴി കണക്റ്റ് ചെയ്തിരിക്കുന്ന ഒരു ബാഹ്യ പ്രോഗ്രാമിലേക്ക് (ഉദാഹരണത്തിന്, [falco-exporter](https://github.com/falcosecurity/falco-exporter)) അലേർട്ട്സ് അയക്കണമെന്നുണ്ടെങ്കിൽ, നിങ്ങൾ [gRPC Configuration section](/docs/grpc/#configuration) എന്നതിന് കീഴിൽ വിവരിച്ചിരിക്കുന്നതുപോലെ `grpc` , `grpc_output` എന്നീ ഓപ്ഷനുകൾ പ്രവർത്തനക്ഷമമാക്കേണ്ടതുണ്ട്.
