Potcoin integration/staging tree
================================

http://www.potcoin.info

Copyright (c) 2009-2013 Bitcoin Developers
Copyright (c) 2014-2014 Potcoin Developers

What is Potcoin?
----------------

Potcoin is a lite version of Bitcoin using scrypt as a proof-of-work algorithm.
 - 40 Second block targets
 - subsidy halves in 840k blocks (~4 years)
 - ~420 million total coins
 - 420 coins per block
 - 107 blocks to retarget difficulty
 - RPC-PORT : 42000
 - P2P-PORT : 4200
 
For more information, as well as an immediately useable, binary version of
the Potcoin client sofware, see http://www.potcoin.info.

P2Pool Settings
----------------
p2pool/networks.py

    potcoin=math.Object(
            PARENT=networks.nets['potcoin'],
            SHARE_PERIOD=10, # seconds target spacing
            NEW_SHARE_PERIOD=10, # seconds target spacing
            CHAIN_LENGTH=24*60*60//10, # shares
            REAL_CHAIN_LENGTH=24*60*60//10, # shares
            TARGET_LOOKBEHIND=200, # shares coinbase maturity
            SPREAD=10, # blocks
            NEW_SPREAD=10, # blocks
            IDENTIFIER='DDA1A1D3B2F68CDD'.decode('hex'),
            PREFIX='A2C3D4D541C11DDD'.decode('hex'),
            P2P_PORT=8420,
            MIN_TARGET=0,
            MAX_TARGET=2**256//2**20 - 1,
            PERSIST=False,
            WORKER_PORT=9420,
            BOOTSTRAP_ADDRS='us-east1.cryptovein.com'.split(' '),
            ANNOUNCE_CHANNEL='#cryptovein',
            VERSION_CHECK=lambda v: True,
        ),




p2pool/bitcoin/networks.py

    potcoin=math.Object(
            P2P_PREFIX='fbc0b6db'.decode('hex'),
            P2P_PORT=4200,
            ADDRESS_VERSION=55,
            RPC_PORT=42000,
            RPC_CHECK=defer.inlineCallbacks(lambda bitcoind: defer.returnValue(
                'potcoinaddress' in (yield bitcoind.rpc_help()) and
                not (yield bitcoind.rpc_getinfo())['testnet']
            )),
            SUBSIDY_FUNC=lambda height: 420*100000000 >> (height + 1)//840000,
            POW_FUNC=lambda data: pack.IntType(256).unpack(__import__('ltc_scrypt').getPoWHash(data)),
            BLOCK_PERIOD=40, # s
            SYMBOL='POT',
            CONF_FILE_FUNC=lambda: os.path.join(os.path.join(os.environ['APPDATA'], 'potcoin') if platform.system() == 'Windows' else os.path.expanduser('~/Library/Application Support/potcoin/') if platform.system() == 'Darwin' else os.path.expanduser('~/.potcoin'), 'potcoin.conf'),
            BLOCK_EXPLORER_URL_PREFIX='http://potchain.potcoin.info/block/',
            ADDRESS_EXPLORER_URL_PREFIX='http://potchain.potcoin.info/address/',
            TX_EXPLORER_URL_PREFIX='http://potchain.potcoin.info/transaction/',
            SANE_TARGET_RANGE=(2**256//1000000000 - 1, 2**256//1000 - 1),
            DUMB_SCRYPT_DIFF=2**16,
            DUST_THRESHOLD=1e8,
        ),



License
-------

Potcoin is released under the terms of the MIT license. See `COPYING` for more
information or see http://opensource.org/licenses/MIT.

Development process
-------------------

Developers work in their own trees, then submit pull requests when they think
their feature or bug fix is ready.

If it is a simple/trivial/non-controversial change, then one of the Potcoin
development team members simply pulls it.

If it is a *more complicated or potentially controversial* change, then the patch
submitter will be asked to start a discussion (if they haven't already) on the
[mailing list](http://sourceforge.net/mailarchive/forum.php?forum_name=bitcoin-development).

The patch will be accepted if there is broad consensus that it is a good thing.
Developers should expect to rework and resubmit patches if the code doesn't
match the project's coding conventions (see `doc/coding.txt`) or are
controversial.

The `master` branch is regularly built and tested, but is not guaranteed to be
completely stable. [Tags](https://github.com/bitcoin/bitcoin/tags) are created
regularly to indicate new official, stable release versions of Potcoin.

Testing
-------

Testing and code review is the bottleneck for development; we get more pull
requests than we can review and test. Please be patient and help out, and
remember this is a security-critical project where any mistake might cost people
lots of money.

### Automated Testing

Developers are strongly encouraged to write unit tests for new code, and to
submit new unit tests for old code.

Unit tests for the core code are in `src/test/`. To compile and run them:

    cd src; make -f makefile.unix test

Unit tests for the GUI code are in `src/qt/test/`. To compile and run them:

    qmake BITCOIN_QT_TEST=1 -o Makefile.test bitcoin-qt.pro
    make -f Makefile.test
    ./potcoin-qt_test

