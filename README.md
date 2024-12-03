# OpenSOD

## Table of Contents
1. [Introduction and Disclaimer](#introduction)
2. [Implementations](#implementations)
3. [File Extensions](#file-extensions)
4. [RFCs](#rfcs)
5. [Copyright](#copyright-statement)

# Introduction

This repo defines the Open Standard OTDR Data (OpenSOD) format. It provides an open-source and standardized way to store and exchange Optical Time Domain Reflectometer (OTDR) measurement data.
Released under the Apache License Version 2.

## Preamble

All OTDRs (Optical Time Delay Reflectometers) on the market today either save their data in the SOR (Standard OTDR Record) format or offer the ability to export to that format.

The current version, which is reportedly defined by SR-4731 (Issue 2), is owned by Ericsson, which doesn't offer a license that allows readers to write compliant software. In fact, the current license explicitly blocks you from creating any "Derivative works", which, when asked for clarification, an Ericsson employee stated includes Open-Source (and even Closed-Source) software.

Fortunately, we are based in the UK, we haven't purchased a license to their IP and have never had any knowledge of the IP's contents, which, thanks to a bit of law the UK adopted from the EU (specifically Directive 91/250/EEC, Article 6), means we are legally allowed to reverse-engineer the API of the standard for the purpose of integrating with systems that use it. 

Unfortunately, from the example SOR files we've seen, every OTDR vendor (including every visualizer and library implementation) has their own unique interpretation of various fields. It's almost as though they, too, have reverse-engineered the standard to avoid licensing issues. 

Now, [Sidney Li](https://morethanfootnotes.blogspot.com/2015/07/the-otdr-optical-time-domain.html?lr=1718804210501) (with some input from others) has already done most of the heavy lifting on reverse-engineering the SOR file format, so we are not taking credit for that. But as we needed to implement a clean C# Library, we figured we'd formalize the current consensus into what we believe could be a suitable open standard.

Readers of this document are encouraged to purchase an appropriate license to [SR-4731 (Issue 2) from Ericsson (formerly Telcordia)](https://telecom-info.njdepot.ericsson.net/site-cgi/ido/docs.cgi?ID=SEARCH&DOCUMENT=SR-4731&#ORD) if they desire to ensure that they have an implementation that is fully compliant with that standard.

What follows is a new standard that supports our independent version of an OTDR data storage format that may (or may not) be compatible with other OTDR data standards.

## Disclaimer

This document is written without knowledge of Ericsson's (or any other rights holder's) intellectual property and has been constructed independently by the Authors. Any compatibility with the Standard OTDR Record as defined by SR-4731 (Issue 2) or any derivative or preceding works thereof is circumstantial and is not guaranteed by the authors. 

Using this document with the intent of being compliant with any formats other than the format defined here is at your own risk.

If readers desire documentation that is fully compatible with the SOR format as defined by SR-4731 (or derivatives), then they should purchase the relevant documents from the rights holders of that standard.

# Implementations
 * [OTDRFile](https://github.com/BaldrAI/OTDRFile) - The official C# implementation of OpenSOD.

# File Extensions

While implementers are welcome to use (and support) any file extension they see fit, it is suggested that the`.SOD` extension is used.

# RFCs
 * [RFC0001](/RFC0001.md) - The initial version maintaining as much interoperability with other standards as possible.

# Copyright Statement
Copyright 2024 BaldrAI Ltd.