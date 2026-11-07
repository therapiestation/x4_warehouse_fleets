# X4 Foundations - Warehouse Fleets Mod

> **Disclaimer – this is a fork.** Warehouse Fleets was created by Manuel Bleichner (mbleichner). All changes made in this repository since the fork (commits after `a96167e` "Allow trading with supply ships") were developed with the help of Claude, Anthropic's AI assistant, and are not part of the original mod. Releases, support and the canonical version remain with the original author.
>
> If the original author requests it, this repository will be removed.

This mod was inspired by the popular [Mules and Warehouses Mod](https://github.com/Misunderstood-Wookiee/Mules-and-Warehouses-Extended) and aims to combine the different mule types into one.

It introduces the WarehouseFleet behaviour, which serves the following purposes:
- Balance cargo levels between warehouses
- Supply factories with resources
- Fetch products from factories
- Sell products to NPC factions
- Supply player-owned build storages
- Fill shortages by buying from NPC factions
- Fill raw material shortages by mining

*Releases will be published on Nexus Mods: https://www.nexusmods.com/x4foundations/mods/939*

## Installation

Just take the whole source folder and put it into your X4 extensions folder.

Required mods:
- [SirNukes Mod Support APIs](https://www.nexusmods.com/x4foundations/mods/503)
- [kuertee's UI Extensions and HUD](https://www.nexusmods.com/x4foundations/mods/552) (note: requires Protected UI Mode to be disabled)

## Intended Use

The mod allows to create a network of warehouses across the galaxy, which will ultimately behave like a single giant shared storage. Arbitrary refineries and factories can be built along this network and will feed from its resources and provide their own products back into the network.

![Warehouse Setup](./WarehouseFleets.drawio.png "Warehouse Setup")

To make this work, every warehouse gets assigned a "WarehouseFleet", composed of a commander ship that uses the WarehouseFleet behaviour and a set of subordinates set to mimic the commander behavior.

## WarehouseFleet Parameters

| Parameter | Description |
| --- | --- |
| Home Warehouse | The home warehouse of this fleet. |
| Targets | A list of sectors or stations, defining the "service area" of the fleet. Ships will only interact with stations contained in the target list. Adding a sector is equivalent to adding all stations of the sector. Mining ships will stay within the sectors defined here. |
| Targets (L) | Targets that are only allowed for L and XL sized ships. Useful for routes through dangerous territory. |
| Targets (M) | Targets that are only allowed for M sized ships. |
| Supply Targets | Target stations that should always get filled to the max and will never provide anything back to the network - useful especially for shipyards. Supply targets also have to be entered in one of the above target settings in order to define which ship class(es) may go there. |
| Prio: Balance Warehouses | Set to high values to generally favor evening out wares between your warehouses (by fill level) over other tasks. |
| Prio: Own Factories | Set to high values to generally favor serving your own (non-warehouse) stations over other tasks: supplying production resources, collecting products and restocking resources for the station's own drones/ammunition. |
| Prio: NPC Trade (Profit) | Set to high values to generally favor buying from / selling to NPC stations over other tasks. Only trades with at least 5% margin that are allowed by the trade rules of the warehouse offers are considered; higher margins (up to 25%) are preferred. |
| Prio: NPC Sell Surplus | Set to high values to generally favor selling surplus wares to NPC stations (see "Sell Surplus: Keep") over other tasks. |
| Prio: Supply Targets | Set to high values to generally favor supply targets (e.g. shipyards) over other tasks. |
| Prio: Build Storages | Set to high values to generally favor filling build storages over other tasks. |
| Prio: Mining | Set to high values to generally favor mining trips over other tasks. Only applicable for mining ships (solid or liquid cargo). |
| Min. Load (% of hold) | Avoids scheduling inefficient trips. No ship will be scheduled that uses less than the specified cargo space. Setting this value too low can cause performance degradation. Setting the value too high might prevent/delay balancing of low volume wares. Ships supplying build storages or restocking a station's drone/ammunition resources may ignore this setting if they fully satisfy the request. Smaller transports can still be added to a trip to the same station that already meets this threshold. |
| Sell Surplus: Keep (%) | Share of each ware's storage target that is never sold as surplus. Everything above it may be sold to NPC stations within the fleet's targets, **ignoring the ware's trade rules** (e.g. player-only wares). Sales never go below the warehouse's own sell price; better-paying buyers are preferred. 100% (default) disables sell-out. |
| NPC Trade: Gate Penalty (%) | Reduces the attractivity of NPC trades (including surplus sales) in other sectors to keep travel distances short. |
| Mining Gate Penalty (%) | Reduces the attractivity of mining in other sectors to keep travel distances short. |

## Global Settings

The global settings can be found in the "Extension Options" via the menu.

| Parameter | Description |
| --- | --- |
| Player Account Threshold | Configures the minimum amount of money that should be left to the player, since buying missing wares from NPC traders can quickly drain your account. |
| Auto-transfer money | Regularly transfer money from station accounts to the player, if at least 10% above the operating budget of the station |
| Cargo emptying routine | Sometimes trades can fail, leaving ships with remaining cargo - this can be resolved in different ways. Check the ingame tooltip for available options. |
| Organize fleets in menu as | "Independent fleets" (default): WarehouseFleet ships will be displayed on the top level of the map/property menus.<br/>"Subordinates of warehouse": fleets will be attached as subordinates under their home warehouses. |


## Warehouses, Trade Wares and Restrictions

The mod uses the normal trade mechanics of the game. So the ships will only be able to transport wares if there are corresponding trade offers set up and their restriction settings are set up appropriately.

The WarehouseFleet ships will also respect buy/sell restrictions to other factions (which is usually not the case in the base game). This is used to control which wares to trade with other factions.

The general setup works like this:
- Create a trade blacklist restricting all trades to only your own faction and set it as default for all your stations
- Add all relevant trade wares in your warehouses (via the station logical overview)
  - Add buy and sell offers (those will inherit the default trade restriction you set up earlier)
  - Set a sell offer to unrestricted to allow your traders to sell off the ware (usually your end products)
  - Set a buy offer to unrestricted to allow your traders to buy the ware from NPC stations (usually the wares you frequently run into shortages)

Be cautious with unrestricted buy offers. Those can drain your account. Use the account threshold setting to control it if necessary.

## Budgets and Money

The player-owned stations don't need any budgets for this mod to work. All trades with NPC factions use the global player account. (I tried to make it work with station accounts, but unfortunately this isn't possible without commander-assignment and this in turn locks the fleet settings so they cannot be changed afterwards - which would be a huge drawback).

If a warehouse has unrestricted sell offers, NPC traders can also buy directly from your warehouse. If this happens, the money goes to the station account. If the station account goes over the operating budget, the surplus will be transferred to the player account.

## Notes

- Any ship might carry up to three different resources at once. Ships sent out to mine will only fetch a single resource.
- Idle ships will dock at their home warehouse and wait for something to do.
- Ships will always begin and end their tasks at the home warehouse. A ship will fly back to the warehouse, even if it has nothing to carry and could in theory fly directly to another station. This is intentional, so that the routes are easier to control and protect. If shortcutting was allowed, the pathing could become unpredictable and ships might fly through hostile regions.
- S class ships will only operate in the local sector. They are generally very inefficient when it comes to longer flights and to cut down on settings, this behavior is hardcoded.
- Ships will only consider a trade if it is profitable. This means they will only sell a ware if your sell offer is cheaper than the buy offer of an NPC station and they will only buy a ware if your buy offer is higher than the sell offer of an NPC station. Usually, you want to set automatic pricing, so your ships will only buy/sell wares if the offer is profitable considering the current market price.
- When using the setting "organize fleets in menu as subordinates of warehouse", the fleets get attached to their warehouse, but this is purely a grouping anchor: the ships behave exactly like independent fleets, their settings stay fully editable, and the station never gives them orders.
- Mining trips that take over 30m will be cancelled to avoid ships getting stuck in depleted ore fields.

## Recommendations

- Add ships of all sizes (S/M/L) to your fleets, so every transport job can be assigned an appropriate-sized ship.
- Restrict traffic between warehouses to L-sized ships. This avoids a lot of pirate/xenon trouble.
- Use the strongest ship of the fleet as commander - if the commander gets killed, you have to redo all of the fleet configuration, because the game doesn't auto-transfer the behavior to another ship of the fleet.
- Check your warehouses from time to time for overworked fleets - if there are no idle ships waiting for work, it might be advisable to add more ships.

## Some Implementation Details

- Only warehouse-to-warehouse transports are balanced by fill level. Your other stations are served by role: consumers of a production resource are filled up to their own storage target, products are collected as long as the warehouse has room, and drone/ammo supply resources are delivered as requested - all independent of the warehouse fill level. This matters for low-volume wares (e.g. metallic microlattice), where warehouses get huge storage targets and thus always have a very low fill ratio.
- To avoid "ping-ponging" of wares, I have implemented two mechanisms:
  - When balancing wares, the cargo will be limited to an amount that perfectly balances the storage levels in both warehouses.
  - Every delivery reduces the attractivity of delivering the same ware in the opposite direction. This effect will decrease over time.
- The base game seems to have a bug causing ships to undock when evaluating modded behaviors. To circumvent unnecessary undock procedures, most of the mod is implemented as a cue instead of an aiscript, so essentially a persistent background process in the game.

## Known Problems

- Setting up the trades of the warehouses is a bit tedious at the moment, because it has to be done one by one in each warehouse. I don't know if there is a good solution to this problem.