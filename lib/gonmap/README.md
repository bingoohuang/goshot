# gonmap

fork from [here](https://github.com/lair-framework/go-nmap)

Nmap XML parsing library for Go

## Saving scan results in an XML format

Extensible Markup Language (XML) is a widely known, tree-structured file format supported by Nmap. Scan results can be
exported or written into an XML file and used for analysis or other additional tasks. This is one of the most preferred
file formats, because all programming languages have very solid libraries for parsing XML and it is widely supported by
third-party security tools.

The following recipe teaches you how to save the scan results in an XML format.

How to do it...
To save the scan results to a file in an XML format, add the option -oX <filename>, as shown in the following command:

```sh
$ nmap -oX <filename> <target>
```

After the scan is finished, the new file containing the results will be written:

```sh
$ nmap -p80 -oX scanme.xml scanme.nmap.org
$ cat scanme.xml
   <?xml version="1.0" encoding="UTF-8"?> 
   <!DOCTYPE nmaprun> 
   <?xml-stylesheet href="file:///usr/local/bin/../share/nmap/nmap.xsl"    
   type="text/xsl"?&gt...
```
