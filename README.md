# Setup

1. Install [Docker Desktop](https://www.docker.com/products/docker-desktop)
2. Install [Algorand sandbox](https://github.com/algorand/sandbox)
3. Add this project folder as bind volume in sandbox `docker-compose.yml` under key `services.algod`:
    ```yml
    volumes:
      - type: bind
        source: <path>
        target: /data
    ```
4. Start sandbox:
    ```txt
    $ ./sandbox up nightly
    ```
5. Install Python virtual environment in project folder:
    ```txt
    $ python -m venv venv
    $ source ./venv/Scripts/activate # Windows
    $ source ./venv/bin/activate # Linux
    ```
6. Use Python interpreter: `./venv/Scripts/python.exe`
    VSCode: `Python: Select Interpreter`

7. Build the contract
   ```txt
   ./build.sh contracts.counter.step_01
   ```
   Make sure `approval.teal` and `clear.teal` files are created under /build/ directory

8. Enter sandbox environment to execute/debug the contract
    ```txt
    ./sandbox enter algod
    ```
9. Get the default accounts list provided by sandbox
    ```txt
    goal account list
    ```

    Result:
    ```txt
    [online]        KNMDERHRZVFTBSEESWOW7XMQO3L5U7TI47YJ5UDMS5HOV5GJG6IB6CHFAE      KNMDERHRZVFTBSEESWOW7XMQO3L5U7TI47YJ5UDMS5HOV5GJG6IB6CHFAE      4000112000000000 microAlgos
    [offline]       PGJAPQVSYNOHLAEWVJO2XIWQF65XLHPE6M5T2RHHBMXRBQYV332L6NJ4GE      PGJAPQVSYNOHLAEWVJO2XIWQF65XLHPE6M5T2RHHBMXRBQYV332L6NJ4GE      1000028000000000 microAlgos
    [offline]       6IPASNKCKFRHOBAK53ALA346C2F266TPBIA2IGILPHJOAWVVNKEGCZRBLM      6IPASNKCKFRHOBAK53ALA346C2F266TPBIA2IGILPHJOAWVVNKEGCZRBLM      4000112000000000 microAlgos
    ```
10. Set $ONE environment variable as the contract owner wuld be used several times during the process
    ```txt
    ONE=KNMDERHRZVFTBSEESWOW7XMQO3L5U7TI47YJ5UDMS5HOV5GJG6IB6CHFAE
    ```
11. Create the application and pass the default arguments
    ```txt
    goal app create --creator $ONE --approval-prog /data/build/approval.teal --clear-prog /data/build/clear.teal --global-byteslices 1 --global-ints 1 --local-byteslices 0 --local-ints 0
    ```
    Output would look like below:
    ```txt
    Created app with app index 1
    ```
12. Run the application by passing the arguments (Increment the argument)
    ```txt
    goal app call --app-id 1 --from $ONE --app-arg "str:inc"
    ```
    `app-id` is the application id from the Step 11.
    `str:inc` tag indicates that how the data we are sending should be interpreted. Example: "int:" - for integer, "addr:" - for addresses. "b64:" for base64 encoded data, "str:"" for string arguments.

    Output:
    ```txt
    Issued transaction from account KNMDERHRZVFTBSEESWOW7XMQO3L5U7TI47YJ5UDMS5HOV5GJG6IB6CHFAE, txid QWTJ3O7K2WWYSQQWVEPTSP6RKXHAR67QBGYAPMCRBHI2YAZMZLPQ (fee 1000)
    Transaction QWTJ3O7K2WWYSQQWVEPTSP6RKXHAR67QBGYAPMCRBHI2YAZMZLPQ still pending as of round 1209
    Transaction QWTJ3O7K2WWYSQQWVEPTSP6RKXHAR67QBGYAPMCRBHI2YAZMZLPQ still pending as of round 1210
    Transaction QWTJ3O7K2WWYSQQWVEPTSP6RKXHAR67QBGYAPMCRBHI2YAZMZLPQ committed in round 1211
    ```
13. Read the inside of the contract storage to check the parameters
    ```txt
    goal app read --global --app-id 1
    ```

    Result: 
    ```json
    {
        "counter": {
            "tt": 2,
            "ui": 1
        },
        "owner": {
            "tb": "SX2D\ufffd\ufffdK0Ȅ\ufffd\ufffdoݐv\ufffd\ufffd~h\ufffd\ufffd\ufffd\ufffdl\ufffdN\ufffd\ufffd\ufffd7\ufffd",
            "tt": 1
        }
    }
    ```

    To read the storage in pretty format:
    ```txt
    goal app read --global --app-id 1 --guess-format
    ```

    Result:
    ```json
    {
        "counter": {
            "tt": 2,
            "ui": 1
        },
        "owner": {
            "tb": "KNMDERHRZVFTBSEESWOW7XMQO3L5U7TI47YJ5UDMS5HOV5GJG6IB6CHFAE",
            "tt": 1
        }
    }
    ```
    `"ui": 1` - counter now has a value of 1.
    
13. Run the application by passing the arguments (Decrement the argument)
    ```txt
    goal app call --app-id 1 --from $ONE --app-arg "str:dec"
    ```
14. Check the storage again to verify the decrement worked.
    ```txt
    goal app read --global --app-id 1 --guess-format
    ```

    Result:
    ```json
    {
        "counter": {
            "tt": 2
        },
        "owner": {
            "tb": "KNMDERHRZVFTBSEESWOW7XMQO3L5U7TI47YJ5UDMS5HOV5GJG6IB6CHFAE",
            "tt": 1
        }
    }
    ```
15. Call the above Decrement operation again to get the following error case:
    ```txt
    Couldn't broadcast tx with algod: HTTP 400 Bad Request: TransactionPool.Remember: transaction LLZB3ENM6WP4MHWYHYLALPR4NX5NRAGEA6RTL7XX23JR2KADWZEQ: logic eval error: - would result negative. Details: pc=90, opcodes=app_global_getintc_1 // 1-
    ```

16. Generate transaction dump file using `--dryrun-dump -o tx.dr` argument. Sandbox comes with a debugger tool called `tealdbg` that can used for debugging the transactions that are failing. 
    ```txt
    goal app call --app-id 4 --from $ONE --app-arg "str:dec" --dryrun-dump -o tx.dr
    ```
    tx stands for transaction, and the file extension .dr for dry file.

17. Initialize a debugger session. The debugger is going to be bound with Chrome debugger.
    ```txt
    tealdbg debug -d tx.dr --listen 0.0.0.0
    ```
    Because it is inside the docker container we have to specify one more argument `listen` with IP address. In our case it is docker container IP address.

    Output:
    ```txt
    2022/05/01 18:58:55 Using proto: future
    2022/05/01 18:58:55 ------------------------------------------------
    2022/05/01 18:58:55 CDT debugger listening on: ws://0.0.0.0:9392/7617e53dc50202480e956a9ce5df84f4881f32bff09ff0299715be104db9ed84
    2022/05/01 18:58:55 Or open in Chrome:
    2022/05/01 18:58:55 devtools://devtools/bundled/js_app.html?experiments=true&v8only=false&ws=0.0.0.0:9392/7617e53dc50202480e956a9ce5df84f4881f32bff09ff0299715be104db9ed84
    2022/05/01 18:58:55 ------------------------------------------------
    2022/05/01 19:02:58 CDT session 7617e53dc50202480e956a9ce5df84f4881f32bff09ff0299715be104db9ed84 closed
    ```

    9392 is the CDT_PORT we have specified in `docker-compose.yml`.

18. Open Chrome session with `chrome://inspect/#devices` address and add `localhost:9392` on Discover network targets -> Configure -> Target discovery settings. Wait for the debugger session to appear under Remote target.

19. Click `inspect` link to open the Chrome debugger with the teal file. Put a breakpoint where the exception occurred and find out the error. The exception occurs as Teal doesn't support a negative number.

    ![Chrome Debugger](https://github.com/bakytnur/algorand-sample-contract/blob/main/chrome_debugger?raw=true)
# Links

- [Official Algorand Smart Contract Guidelines](https://developer.algorand.org/docs/get-details/dapps/avm/teal/guidelines/)
- [PyTeal Documentation](https://pyteal.readthedocs.io/en/latest/index.html)
- [Algorand DevRel Example Contracts](https://github.com/algorand/smart-contracts)
