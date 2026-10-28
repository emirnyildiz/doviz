# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

A single self-contained page, `doviz-altin.html`: a Turkish-language currency and gold rate viewer and converter (TRY, USD, EUR, and gram/çeyrek/yarım/tam/ons gold). HTML, CSS and vanilla JS all live in that one file. There is no build step, package manager, linter or test suite. Not a git repository.

## Running

Open `doviz-altin.html` directly in a browser (`start doviz-altin.html` in PowerShell). It needs internet access to fetch rates.

## Architecture

- **Data source:** `fawazahmed0/exchange-api` (no key, CORS-enabled, updated **daily**, so not live market rates). `latestUrls` holds the jsDelivr endpoint with a `currency-api.pages.dev` fallback, and `fetchFirst` tries them in order. The previous day's file (`datedUrls(prevDay(date))`) feeds the % change shown on the cards.
- **Rate model:** everything is USD-based. `pick()` reduces the response to `{ try, eur, xau }`, where `xau` means troy ounces per 1 USD. `tl(code, r)` turns any unit into its TRY value, and conversion is `amount * tl(from) / tl(to)`. Gold coins use 22 karat purity (`AYAR22 = 0.916`) and standard weights (çeyrek 1.754 g, yarım 3.508 g, tam 7.016 g), so they show melt value, not jeweller prices.
- **Config tables:** `UNITS` (converter units, with labels, badges and display decimals) and `CARDS` (top rate cards, each with a `val(r)` getter and the `from`/`to` pair it opens in the converter). To add a unit, extend `UNITS` and add a case in `tl()`.
- **Rendering:** `renderAll()` re-renders the cards and the converter from the globals `rates`, `prevRates` and `dataDate` by rebuilding `innerHTML`. Clicks are handled by delegated listeners on `#cards`, `#list` and `#chips`.
- **Persistence:** `store` wraps `localStorage` in try/catch. Keys are `kur-cache` (the last fetched rates, shown first and used offline), `kur-prefs` (amount, from, to) and `kur-theme`.
- **Theming:** colour tokens are CSS variables on `:root`. Dark values are defined twice: under `prefers-color-scheme` (guarded by `:root:not([data-theme="light"])`) and under `:root[data-theme="dark"]`. Keep both blocks in sync.
- **Number format:** display uses `Intl.NumberFormat('tr-TR')` through `fmt()`. `parseAmount()` accepts Turkish input (`1.250,50`).
