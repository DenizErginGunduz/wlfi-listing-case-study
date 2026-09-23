# Evidence trail | Five headline claims

Each step links a public record. On-chain claims can be re-read at the stated block with any archive-capable Ethereum RPC; the commands below use a public endpoint and are pinned to historical blocks, never `latest`.

```bash
RPC=https://eth.drpc.org
TOKEN=0xdA5e1988097297dCdc1f90D4dFE7909e847CBeF6   # WLFI proxy
SAFE=0x5be9a4959308A0D0c7bC0870E319314d8D957dBB    # owner multisig
CUTOFF=0x162B031                                    # block 23,244,849 (29 Aug 2025, 06:00 UTC)
call(){ curl -s -X POST -H 'content-type: application/json' \
  --data "{\"jsonrpc\":\"2.0\",\"id\":1,\"method\":\"eth_call\",\"params\":[{\"to\":\"$1\",\"data\":\"$2\"${4:+,\"from\":\"$4\"}},\"$3\"]}" $RPC; echo; }
```

## 1. The cutoff precedes the first spot-listing notice

- **HTX notice.** HTX published [its WLFI spot-listing notice](https://www.htx.com/support/55010697762922) at **06:28:15 UTC on 29 Aug 2025**. The time comes from the exchange's announcement API (`showTime` = 1756448895104 ms).
- **KuCoin notice.** KuCoin's [listing notice](https://www.kucoin.com/announcement/en-world-liberty-financial-wlfi-gets-listed-on-kucoin-world-premiere) followed at 07:18:02 UTC (`cTime` = 1756451882000).
- **The cutoff.** The cutoff is 06:00:00 UTC. Ethereum block 23,244,849 is the last block before it.
- **Pre-market markets.** Pre-market derivatives were trading from 23 August. They are treated as market data, not as the listing event.

## 2. What was already possible in V1, and what V2 added

| Power | V1 ([source](https://etherscan.io/address/0x3722359be0bfebb541bc98adfe1250cd901a706c#code), 29 Sep 2024) | V2 ([source](https://etherscan.io/address/0x0959a6eaea3c23148fe69ddd703c277bc6ad79cc#code), 24 Aug 2025) |
| --- | --- | --- |
| Pause transfers | `pause()`: owner or guardian | `ownerPause`, `guardianPause` |
| Unpause | `unPause()`: owner | `ownerUnpause`: owner only; no guardian unpause |
| Blacklist an address | none | `ownerSetBlacklistStatus`, `guardianSetBlacklistStatus` (set or clear) |
| Remove balances | `adminBurn`: owner burns from allow-listed accounts | removed |
| Move balances | none | `ownerReallocateFrom`: burn from one address, mint to another |
| Guardian keys appointed | none (no `SetGuardian` event from block 0 until the V2 upgrade) | `0x407f…c178`, in the upgrade transaction |

In `WorldLibertyFinancialV2.sol`, both legs of a reassignment pass through `_update`. That function rejects a blacklisted caller, source or recipient, rejects any transfer while paused, and after trading start rejects non-activated sale wallets. The owner can change each of those conditions.

Simulated at block 23,479,243 (`0x16643CB`), while V2 was still deployed:

```bash
pad(){ a=${1#0x}; printf '%064s' "$(echo $a | tr A-F a-f)" | tr ' ' 0; }
BL=0x5AB26169051d0D96217949ADb91E86e51a5FDA74      # an address blacklisted on 5 Sep 2025
OK=0x5533e41b3d71823ee5bcfc22a472e83fcaaa5378      # a non-blacklisted holder (Uniswap pool)
DEAD=0x000000000000000000000000000000000000dEaD
call $TOKEN 0x7df9a674$(pad $OK)$(pad $DEAD)$(pad 1) 0x16643CB $SAFE   # succeeds: "result":"0x"
call $TOKEN 0x7df9a674$(pad $BL)$(pad $DEAD)$(pad 1) 0x16643CB $SAFE   # reverts: 0xffa4e618 = Blacklisted(address)
```

## 3. 564.25 million WLFI reassigned in the upgrade transaction

The [upgrade transaction](https://etherscan.io/tx/0xaa6f0c426ec9ce530a4e68d9eeff4698049141a2c1391d08a90b2ef6a3d7a506#eventlog) (block 23,207,432, 00:42:11 UTC, 24 Aug 2025) emits three `Reallocated` events (topic `0x7010a46c…71004006`):

| From | To | WLFI |
| --- | --- | ---: |
| `0xae627a70208d6785ed703326d35221237667ae46` | `0xf32e3596f555546acc4ad6ef67e1abf36b134748` | 562,500,000 |
| `0x7ae6f4c47694acab7575b8406f0108062193da84` | `0x45842d6da48f0d0cf3e7f4bf7db440ab803b36b1` | 1,746,076.0754 |
| `0x119ca458c5f91d3815e5f8a29633d709569b4eb3` | owner multisig | 1 |
| **Total** | | **564,246,077.0754** |

No public statement explaining these transfers was found before the cutoff. The auditor's report on the V2 upgrade is dated 5 September 2025, after the cutoff ([report](https://github.com/Cyfrin/cyfrin-audit-reports/raw/main/reports/2025-09-05-cyfrin-wlf-v2.1.pdf)).

## 4. Who held the powers at the cutoff

```bash
call $SAFE  0xe75235b8 $CUTOFF        # getThreshold()  -> 3
call $SAFE  0xa0e67e2b $CUTOFF        # getOwners()     -> five addresses, including 0x407f66af…9705c178
call $TOKEN 0x8da5cb5b $CUTOFF        # owner()         -> the multisig
call $TOKEN 0x0c68ba21$(pad 0x407f66afb4f9876637accc3246099a2f9705c178) $CUTOFF   # isGuardian -> true
call $TOKEN 0x70a08231$(pad $SAFE) $CUTOFF   # balanceOf(multisig) -> 32,590,415,020.72 WLFI
```

The multisig also owned the proxy admin (`0x7f533afa9e994b7e44673188034f9ce52886a0c0`), so it could upgrade the token without a timelock.

## 5. The public-sale buyers' initial-unlock ceiling and the ownership gap

**Public-sale buyers' initial unlock.** Every input below is dated before the cutoff.

| Input | Value | Source (date) |
| --- | --- | --- |
| First sale round | $300 M | [Issuer release](https://www.businesswire.com/news/home/20250317855322/en/Trump-Inspired-World-Liberty-Financial-Closes-%24550-Million-Across-Token-Sales) (17 Mar 2025) |
| Second sale round | $250 M | Same release |
| Round prices | $0.015 and $0.05 | [Issuer post on X](https://x.com/worldlibertyfi/status/1959004569868108277) (22 Aug 2025, 21:27 UTC); also [CoinDesk](https://www.coindesk.com/markets/2024/10/15/trump-familys-world-liberty-finance-token-sale-appears-to-go-live-selling-220m-wlfi-in-minutes) (15 Oct 2024) and [The Block](https://www.theblock.co/post/346331/trump-backed-world-liberty-financial-concludes-550-million-public-token-sale) (14 Mar 2025) |
| Initial unlock | 20 % of the WLFI each wallet bought in those rounds | Same issuer post: "20% of WLFI purchased by each wallet in the $0.015 and $0.05 rounds will unlock" |

The calculation:
- $300 M / $0.015 = **20 B** tokens.
- $250 M / $0.05 = **5 B** tokens.
- 20 % × 25 B = **5.0 B**.

The token counts match the contemporaneous reports: 20 B tokens at $0.015 (CoinDesk) and a further 5 B at $0.05 (The Block).

**What the figure bounds.** 5.0 B is a ceiling on what public-sale buyers could claim at launch. It is not a cap on circulating or tradable supply, because issuer, affiliate and liquidity holdings could also become transferable. It is a ceiling rather than an estimate because no source showed which buyers received the tranche.

The issuer's [MiCA white paper](https://static.worldlibertyfinancial.com/docs/mica-whitepaper.pdf), published after the cutoff, restates the $300 M / $250 M split. It is retrospective corroboration only.

**Beneficial ownership.** DT Marks DEFI LLC's stake in WLF Holdco LLC was reported at 75 % at the end of 2024, about 60 % in early 2025 and 40 % in June 2025 ([Crypto Briefing, citing Forbes](https://cryptobriefing.com/world-liberty-stake-reduction/)). No public source identified the holders of the remainder.

**After the cutoff (retrospective confirmation only).** The issuer's [1 September launch supply post](https://web.archive.org/web/20260222163046/https://medium.com/@wlfi/understanding-the-wlfi-circulating-supply-at-launch-061f904e6923) listed 24,669,070,265 tokens as circulating. Of these, 4,004,600,000 were the public-sale unlock, exactly 20 % of a 20.023 B public-sale category. The rest were issuer, affiliate and liquidity tokens, which is why circulating supply exceeded the public-sale ceiling. This was published after the cutoff and does not inform the decision.

[Return to overview](README.md) · [Method and limits](METHOD_AND_LIMITS.md)
