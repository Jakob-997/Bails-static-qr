# Static QR transfer

Open **CipherStick** > **QR transfer**, or the **QR transfer** application.
This tool uses one static QR per transfer. It does not use BBQr or UR.
The main menu offers **Verify Receiving Address**, **Receive PSBT**, **Send PSBT**,
and **Wallet Tools**. Each scanning action accepts only its selected type.

## Requirements

Use the normal Bails installation on Tails, with Bitcoin Core running.
The tool uses `bitcoin-cli`, Zenity, `zbarcam` from `zbar-tools`, `xdg-open`,
and Tails' GTK 3 Python bindings for the clipboard,
Python 3, and Tails' `python3-qrcode` library and `python3-qr` command. It uses the
library's default PNG renderer without selecting PIL or another image factory.
It installs nothing.
Options such as `-regtest` and `-datadir=/path` can be passed to `bails-qr`;
they are forwarded to `bitcoin-cli`.

## Transfer the watch-only wallet

1. On the CryoStick, open the signing wallet in Bitcoin Core.

2. Choose **Wallet Tools** > **Export Watch-Only Wallet** and select that loaded wallet.
   The tool requests all public descriptors using `listdescriptors false`,
   compresses them with gzip, and opens a static QR image. Private keys are not
   requested. Descriptor timestamps, receiving/change settings, ranges, and next
   address indexes are included. Labels and saved payment requests are not.

3. On the CipherStick, choose **Wallet Tools** > **Import Watch-Only Wallet**.
   Hold the QR steady in view of the camera. Close the camera window to cancel;
   scanning times out after
   three minutes.

4. The tool creates a blank wallet named **[Watch-only] <original wallet name>**,
   with private keys disabled, and imports the descriptors. It creates the wallet
   in Core's configured wallet directory. An existing wallet with that name is
   not reused. Slashes in the original name become underscores.

5. Wait for Core's import and blockchain rescan to finish, then select the wallet
   in Core. The scan starts from the exported descriptor timestamps and can take
   time. Historical blocks must be available; a pruned node may need additional
   blocks. If any descriptor fails, the tool reports the failure; the new wallet
   may be incomplete. Old *wallet.dat* QR exports are no longer accepted.

## Transfer a PSBT

1. Copy an unsigned or signed PSBT as text from Bitcoin Core.

2. Choose **Send PSBT**, paste the text into the box, and click **OK**. Core checks that it decodes.
   The static QR contains its Base64 representation, which Sparrow can scan.

3. On the other computer, choose **Receive PSBT**. Core checks that the transaction
   decodes, then the tool shows its Base64 text. Click **Copy**, or select the
   text and press **Ctrl+C**.

4. In Core, select your wallet, then choose **File** > **Load PSBT from clipboard**. Review the transaction
   before signing or broadcasting. The QR tool does neither.
   **Copy** leaves the text window open. **Close** exits immediately; the tool
   never waits for Core or for a confirmation that the PSBT was loaded.

Scanning also accepts the gzip-compressed PSBT format in the manual
[CryoStick instructions](CRYOSTICK.md). It does not accept Sparrow's UR or
BBQr QR exports; copy the PSBT as Base64 text in Sparrow for the return trip.

## Check a receive address

Choose **Verify Receiving Address** and scan an address or a `bitcoin:` payment URI. The tool
checks Core's `ismine` result across the open wallets automatically. Open your
signing wallet in Core first; no wallet selection is needed for this check.
An owned address shows **You own this address**. An address that none of the open
wallets recognize gets a warning to confirm it again on the offline signing
wallet before sending funds. Bech32 addresses are displayed in lowercase;
case-sensitive Base58 addresses keep their original case. Payment URI amounts
and other parameters are not used.

## Transfer files when a QR is too large or no camera is available

A binary QR can hold at most 2,953 bytes at the lowest error correction level.
Descriptor transfers use compressed bytes; PSBT transfers use Base64 text, which
leaves room for at most 2,214 PSBT bytes. Real cameras may not read the densest
codes reliably. Descriptor sets can exceed these limits even with compression.

If a transfer is too large, the tool offers to save the original file.
Use a new filename. To transfer without a camera:

1. Use the saved public descriptor JSON, or save the PSBT in Core.

2. Copy the exported file to a USB stick or microSD card, move it to the
   receiving computer, and copy it there.

3. Import the saved JSON's `descriptors` array using Core's `importdescriptors` RPC in a blank
   wallet with private keys disabled, or load the PSBT from file in Core.

The QR opens in Tails' normal image viewer without a separate instruction popup.
Close the image after scanning. The viewer controls window size and zoom.

The QR tool keeps working files in `/tmp` with private permissions and removes
them when it exits normally. Saved files are written only where you choose;
existing files are not overwritten.

## Validation

Run `python3 tests/bails-qr.py` with `qrcode`, Pillow, and `zxing-cpp`
available in the test environment. The QR tests use the upstream generator
and decode its PNG output independently.

The Windows development checks cover QR image round trips with an independent
decoder, compressed public descriptors, raw oneshot scan output without an appended
newline, size limits, PSBT RPC validation, address selection, cancellation,
and file saving. Core and desktop
calls are simulated in those checks. They do not replace a Tails test.
An isolated Core 32.0rc4 check also verified single-signature and 3-of-7 descriptor
round trips, matching next receiving/change addresses, and rejection of private
descriptors by the destination wallet. A Linux GTK check verified Copy and Close.

Before using real funds, test on the supported Tails release with two computers:
export and import a fresh descriptor wallet, compare its next receiving and change addresses,
transfer an unsigned PSBT, sign it in Core, transfer it back, and check an address.
Also exercise camera cancellation, an unreadable QR, an oversized export,
an invalid PSBT, a missing dependency, and a save destination that already exists.
