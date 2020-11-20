---
layout: post
title: Removing Date from Filename 
categories: [Batchfile,script]
---

This project required a batchfile to be used in lieu of Powershell, so it is not as elegant as the one-line `(Get-Date).AddDays(-1)` solution Powershell offers.

Files came in to a repository each day, prefixed with the previous day's two digit date. This allowed multiple days worth of files to be stored in the repository, but the dynamic prefix needed to be removed so it would work with Apache Pig ETL. 

This script reads in the current date - parsed into month, day, and year - and determines the previous date based on whether it is the first of the month (and if so, which month).

[Link to script in Github](https://github.com/Lwillio/scripts/blob/main/previousDateFindandRemove.bat)

```batchfile
set DD=%date:~7,2%
set MM=%date:~4,2%
set YY=%date:~10,2%
set currdate=%YY%-%MM%-%DD%
echo %currdate%
set offset=01 

{...}

if NOT %DD%=="01" goto :sameMonth
if %DD%=="01" if %MM%=="01" goto :longMonth
if %DD%=="01" if %MM%=="02" goto :longMonth
if %DD%=="01" if %MM%=="03" goto :feb
if %DD%=="01" if %MM%=="04" goto :longMonth
if %DD%=="01" if %MM%=="05" goto :shortMonth
if %DD%=="01" if %MM%=="06" goto :longMonth
if %DD%=="01" if %MM%=="07" goto :shortMonth
if %DD%=="01" if %MM%=="08" goto :longMonth
if %DD%=="01" if %MM%=="09" goto :longMonth
if %DD%=="01" if %MM%=="10" goto :shortMonth
if %DD%=="01" if %MM%=="11" goto :longMonth
if %DD%=="01" if %MM%=="12" goto :shortMonth

```
