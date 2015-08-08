strongswan vici golang client
=============================
[![Build Status](https://travis-ci.org/bronze1man/goStrongswanVici.svg)](https://travis-ci.org/bronze1man/goStrongswanVici)
[![GoDoc](https://godoc.org/github.com/bronze1man/goStrongswanVici?status.svg)](https://godoc.org/github.com/bronze1man/goStrongswanVici)
[![docs examples](https://sourcegraph.com/api/repos/github.com/bronze1man/goStrongswanVici/badges/docs-examples.png)](https://sourcegraph.com/github.com/bronze1man/goStrongswanVici)
[![Total views](https://sourcegraph.com/api/repos/github.com/bronze1man/goStrongswanVici/counters/views.png)](https://sourcegraph.com/github.com/bronze1man/goStrongswanVici)
[![GitHub issues](https://img.shields.io/github/issues/bronze1man/goStrongswanVici.svg)](https://github.com/bronze1man/goStrongswanVici/issues)
[![GitHub stars](https://img.shields.io/github/stars/bronze1man/goStrongswanVici.svg)](https://github.com/bronze1man/goStrongswanVici/stargazers)
[![GitHub forks](https://img.shields.io/github/forks/bronze1man/goStrongswanVici.svg)](https://github.com/bronze1man/goStrongswanVici/network)
[![MIT License](http://img.shields.io/badge/license-MIT-blue.svg?style=flat-square)](https://github.com/bronze1man/goStrongswanVici/blob/master/LICENSE)

a golang implement of strongswan vici plugin client.

### document
* http://godoc.org/github.com/bronze1man/goStrongswanVici
* https://github.com/strongswan/strongswan/tree/master/src/libcharon/plugins/vici

### Implemented command list
* version()
* list-sas()
* terminate()
* load-conn()
* unload-conn()
* load-shared()

If you need some commands, but it is not here .you can implement yourself, and send a pull request to this project.

### example
```go
package main

import (
	"fmt"
	"github.com/bronze1man/goStrongswanVici"
)

func main(){
    // create a client.
	client, err := goStrongswanVici.NewClientConnFromDefaultSocket()
	if err != nil {
		panic(err)
	}
	defer client.Close()

	// get strongswan version
	v, err := client.Version()
	if err != nil {
		panic(err)
	}
	fmt.Printf("%#v\n", v)

	childConfMap := make(map[string]goStrongswanVici.ChildSAConf)
        childSAConf := goStrongswanVici.ChildSAConf{
                Local_ts:      []string{"10.10.59.0/24"},
                Remote_ts:     []string{"10.10.40.0/24"},
                ESPProposals:  []string{"aes256-sha256-modp2048"},
                StartAction:   "trap",
                Mode:          "tunnel",
                ReqID:         "10",
                RekeyTime:     "10m",
                InstallPolicy: "no",
        }
        childConfMap["test-conn"] = childSAConf

        localAuthConf := goStrongswanVici.AuthConf{
                AuthMethod: "psk",
        }
        remoteAuthConf := goStrongswanVici.AuthConf{
                AuthMethod: "psk",
        }

        ikeConf := goStrongswanVici.IKEConf{
                LocalAddrs:  []string{"192.168.198.10"},
                RemoteAddrs: []string{"192.168.198.11"},
                Proposals:   []string{"aes256-sha256-modp2048"},
                Version:     "1",
                LocalAuth:   localAuthConf,
                RemoteAuth:  remoteAuthConf,
                Children:    childConfMap,
                Encap:       "no",
        }

        conn := &goStrongswanVici.Connection{
                ConnConf: ikeConf,
        }

	//load connenction information into strongswan
        err = client.LoadConn(conn)
        if err != nil {
                fmt.Printf("error loading connection: %v")
                panic(err)
        }

	sharedKey := &goStrongswanVici.Key{
                Typ:    "IKE",
                Data:   "this is the key",
                Owners: []string{"192.168.198.10"}, //IP of the remote host
        }

	//load shared key into strongswan
        err = client.LoadShared(sharedKey)
        if err != nil {
                fmt.Printf("error returned from loadsharedkey \n")
                panic(err)
        }


	// get all conns info from strongswan
	connInfo, err := client.ListAllVpnConnInfo()
	if err != nil {
		panic(err)
	}
	fmt.Printf("found %d connections. \n", len(connInfo))

	//unload connection from strongswan
	unloadConnReq := &goStrongswanVici.UnloadConnRequest{
			Name: "test-conn",
			}
	err = client.UnloadConn(unloadConnReq)
	if err != nil {
		panic(error)
	}

	// kill all conns in strongswan.
	for _, info := range connInfo {
		fmt.Printf("kill connection id %s\n", info.Uniqueid)
		err = client.Terminate(&goStrongswanVici.TerminateRequest{
			Ike_id: info.Uniqueid,
		})
		if err != nil {
			panic(err)
		}
	}
}
```
