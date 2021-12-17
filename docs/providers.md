# Providers
### ServicesProviders

Components: `loader` `confetti`

Logic: `connection` `wallet` `accounts` `SPLTokenList` `store` `coingecko` `meta`

### AppLayout

-Pages / -Sections

## Connection Provider
* Has a defined interface for the `ConnectionConfig`. Which have a connection prop + an specific endpoint (for the Solana-API) + an ENV definition (mainnet, devnet, etc) + set function for the endpoint + Token Array.

```js
//
export interface TokenInfo {
  readonly chainId: number;
  readonly address: string;
  readonly name: string;
  readonly decimals: number;
  readonly symbol: string;
  readonly logoURI?: string;
  readonly tags?: string[];
  readonly extensions?: tokenExtensions;
}
```
* To create a `ConnectionContext` that heritates the ConnectionConfig structure and defines some base values.

```js 
//
const ConnectionContex = React.createContext<ConnectionConfig>({
  endpoint: DEFAULT,
  setEndpoint: () => { },
  connection: new Connection(DEFAULT, 'recent'),
  env: ENDPOINTS[0].name,
  tokens: [],
  tokenMap: new Map<string, TokenInfo>(),
});
```
* As a provider is expected to receive children that will then on the context provider.
* Initially gets the actual connected network and search it into an ENDPOINTS Array, in order to get the API endpoint.
* Then makes a `useMemo` instance for connection, which will allow to generate a new connection whenever the endpoint changes (eg: switching between mainnet and devnet)
* Also gets the ENV name.
* Creates two states: to store tokens(`TokenInfo Array`) and a `key/Value Map` for those tokens.
* Creates three UseEffects: 
  1. For Fetching the Token files whenever the ENV is new or changes.
  2. To create a `keyPair` and remove the accountChangelistener from a previous account whenever the connection is new or changes.
  3. To create an `id` and remove the slotChangelistener from a previous connection whenever its is new or changes.
* This Connection Provider returns a ConnectionContext.Provider which brings the `ConnectionConfig`.

The rest de `connection.tsx` file provides some methods to:
* SendTransactionsWithManualRetry.
* SendTransactions.
* SendTransaction(individual)
* SendTransactionWithManualRetry(individual)
* SendSignedTransaction.
* awaitTransactionSignatureConfirmation(for Phantom Wallet interaction)

Contexts are most likely global states(following the `setState statement`), Contexts work as a global states(following the `setState behaviour`), where every child of the father context component heritates the info as props from the parent and so on every child of the father context component heritates the info as props from the parent and so on.

```js
<ConnectionProvider>
      <WalletProvider>
        <AccountsProvider>
          <SPLTokenListProvider>
            <CoingeckoProvider>
              <StoreProvider
                ownerAddress={process.env.NEXT_PUBLIC_STORE_OWNER_ADDRESS}
                storeAddress={process.env.NEXT_PUBLIC_STORE_ADDRESS}
              >
                <MetaProvider>
                  <LoaderProvider>
                    <ConfettiProvider>
                      <AppLayout>{children}</AppLayout>
```
`useMemo` hook avoids getting the full component re-render when a certain state changes, the memo function will only be executed on a state change of the memo params have changed regarding the last saved value.
Most likely is to been used with slow functions.

## Wallet Provider
* Creates a context to manage the Wallet Modal(defines a visible attribute and a `setVisible`)
* Wallet Modal(component) use:
  1. The Solana Wallet adapter to get all  the actual wallets, the current selected Wallet, and a method to select a new one.
  2. The `WalletContext` to set the Modal visibility.
  3. A `Callback` to manage how the Wallets disappear when the cancel button is pressed on the modal.
  4. a Memo to get the `phantomWallet` info without re-render the whole `WalletModal` component.

* `WalletModalProvider` shares: publicKey, connected(state) and Wallet visible(state) with children nodes; and notifies if the provider was disconnected or connected to an specific Wallet set by the user.
* `WalletProvider` is the main provider in charge to connect with _PhantomProvider, SolFlareWallet, LedgerWallet_, etc.

## Accounts Provider.

## Meta Provider.


To create a store, you must first derive the store ID given your public address. The Metaplex devs have already created
an environment variable for you to utilize - `REACT_APP_STORE_OWNER_ADDRESS_ADDRESS` - which you should set to be your
wallet public address. To do this, you can create a `.env` file in `packages/web`, and set
`REACT_APP_STORE_OWNER_ADDRESS_ADDRESS` to be your wallet public address in there.
-------------------------------------------

```
REACT_APP_STORE_OWNER_ADDRESS_ADDRESS=YOUR_PUBLIC_WALLET_ADDRESS
```

### Create Your Store

After creating your store ID, you may now create your store. The Metaplex platform has many helper methods to help you
to create your store. To create your store, you can use the `saveAdmin` method (`packages/web/src/actions/saveAdmin`)
The easiest way to do this would be to either create a script or render a button locally to click to call this method.
Please look at the function parameters of `saveAdmin` to see what parameters you would like to pass in:

```js
saveAdmin(connection, wallet, false, [])
```

If you opted to create a button or something to click to call this method, here are some small snippets:

```js
// These are hooks you should insert at the top of the component your rendering your button in
const { wallet } = useWallet();
const connection = useConnection();
```

```js
// The button to render somewhere for you to click
<Button onClick={async () => {
        try {
          await saveAdmin(connection, wallet, false, [])
        } catch (e) {
          console.error(e);
        }
}}>CREATE STORE</Button>
```

You will be required to confirm your transactions if you decided to put a button or something to click. After clicking
the button, make sure you don't browse anywhere else or close the website.

### Adding Your Information

After creating your store, you must also insert your wallet public key and information in `userNames.json` at
`packages/web/src/config/userNames.json`. Make sure you follow the same format as the other objects in this file.

### Accessing the Admin Panel

After creating your store, you can now access `YOUR_URL/#/admin`. This is where you can edit your store and add
whitelisted creators. Add yourself if you need to or make it a public store, so anyone can create NFTs in your store.
Remember to click save after making your changes.
