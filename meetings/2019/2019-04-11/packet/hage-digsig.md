CH: 3/12/2019 - Digital Signature Draft Proposal for Discussion

## 6. Recommendations

### 6.11. Security

#### 6.11.1. Digital Signatures

Digital signatures are ubuquitous within the current software development
community, but currently are virtually unused in securing election data.
A new election system developed for the future must incorporate digital
signatures to insure the integrity and validity of election data.

Manufacturers that work to insure the quality of thier products address
quality within the whole supply chain. With election data security,
it is likewise important to make use of digital signatures to insure
integrity, validity, and traceability across the whole chain of source
input data, derived data, configuration files, software, and output data.

Besides digital signatures used to authenticate and secure election
data, a primary basis of data security should be from posting the complete
chain of non-confidential data on a network-accessible server with open
access and subject to public scrutiny.

The following requirements are guidelines for using digital signatures
for data and software:

* All Election-related data transferred between computers must have a
  digital signature that can be used to validate the integrity and origin
  of exchanged data. (This includes data archived for backup.)

* The usage of creating digital signatures and how to authenticate
  signatures must be in a documented process.

* The software used to create digital signatures should be open-source
  and use industry standard signature formats. Instructions should be
  provided to allow users to utilize open-source software to validate
  digital signatures.

* The secret keys used to create digital signatures should be stored
  in a hardware security module. (A TPM chip or plug-in cryptographic
  device.) Digital signature keys used with a special "certified" status
  should be stored in a removable hardware security module that has
  controlled access.

* After a set of data processing has completed, it is recommended that
  the newly created/updated collection of data be secured with a digital
  signature.

* A single digital signature should be able to secure a set of data files.
  By validating a signature, the authentication can then be applied to a
  collection of data.

* Additional "endorsement" digital signatures should be enabled to provide
  added authentication and special meaning.

    * Certain important data that is currently "certified", e.g. candidate lists or
      results, can have a special added signature that adds a certified status.
      Critical data files, such as ballot definition files used for Ballot Marking
      Devices or vote counting software, should also require a certified status to be
      used by these critical election application components.

    * A "notarization" digital signature could be added as a record that a specific
      set of data was posted at a certain time with verified digital signatures.
      For example, the Secretary of State could operate a "notary" server,
      that validates the primary digital signature, and attaches a second
      signature. A public record of the notary server signatures issued
      can be used to confirm the time and content of digitally notarized data.

    * Data exported from one infosystem might have a digital signature created
      by the hardware in the source system, then imported into another
      infosystem. After importing, an additional digital signature created
      by the importing system hardware could be added. (The importing system
      might convert the original "primary" signature to a secondary/endorsement
      with the new signature created by the importing system acting as a new
      "primary" signature.

* All software used should have digital signatures attached and validated.

    * Computer systems must use a secure boot to authenticate software initially
      loaded on startup, and to authenticate the base operating system and
      secured data collection. Ideally, the secure boot would require a special
      digital signature, e.g. from a state voting certification, and the
      signature required could not be changed without replacing the hardware.

    * Software packaged into containers should have digital signatures associated
      with the container file and validated at least upon system startup.

    * Software collections (e.g. the complete set of source and library files
      used by an application should have an associated hash for each file
      (digital fingerprint) included in an overall signature, so changes
      and versioning of individual files can be tracked.

    * Configuration files used by the operating system or used by application
      components could be included in the software signatures, or in a separate
      signature associated with a data collection.

* Critical security applications should validate all input data, either at
  startup, or while reading data.

    * Applications packaged into containers should be designed to have
      input data (including configuration files) presented as read-only
      files of digitally signed data.

    * Critical applications should be configured to require a special
      endorsing digital signature indicating certified data.

    * New output data, or updated/replacement files should be signed upon
      completion of the component processing.

* Log files should be created by the originating application, or in post
  processing by a controlling process management app, that includes the
  hash (digital fingerpring) of input/configuration files, software collection,
  and new output data created. These log files should be included in the
  newly signed output data.

    * Log files with hash codes tie a particular set of input data and
      software to a collection of new output data.

    * When replacing updated data, the prior hash codes and signatures
      can be included to track revisions.

    * Input, configuration, software, and output file hash codes provide
      a secure chain of custody on all data and facilitate auditing and
      versioning of data.

* Data files should be stored/posted in a manner that facilitates version
  control and difference analysis, tracked with digital signatures.

    * Data stored as text files with line breaks that facilitate line-by-line
      comparison and differences is preferred.

    * Dividing data according to when changes are made is preferred. For example,
      the list of contests in an election normally does not change, while
      candidate filing data changes until filing closes and the final candidate
      list is determined, so separating contests from candidates allows tracking
      of changes in either independently.

    * Database, etc. files that combine data collections into continuously
      changing files should be considered secondary data derived from the
      digitally signed source. Validation software must be included to
      validate database, etc. file content with a primary text-based master
      data file. (Sometimes binary database files can change values even
      when data inside is unchanged, and individual differences cannot be
      tracked in the binary database file.)

## Appendix X - Sample Digital Signature Protocol

This appendix contains a sample implementation of the digital signature
recommended requirements. This implementation is used in the OSVTAC
sample data and application repositories as a demonstration. It can be
used as-is or as the basis to develop an alternative/improved protocol.

### X.1 - Data file collections

A collection of data files is stored in a file system, and can be signed as
a set with a single digital signature. All files in a directory can be considered
to be the collection, or a subset of files in a directory can be in a collection
with an explicit list of file names defining the subset. Files in a directory
tree can be in a single collection signed at the top level, or each subdirectory
can have independent signatures.

#### X.1.1 - Compressed Archive Files

A data collection representing all or a subset of files in a file directory
or directory subtree can be packaged into a `.zip` or other archive format.
The `.zip` file should include the same hash and signature files that would
otherwise apply to files stored in a file system directory. The `.zip` file
containing signed data can be made available to download a data collection,
and can also be used directly by application software.

A signature on the top-level `.zip` file is not required (all individual files
inside should be digitally signed), but the binary zip file could be included
in a separate collection of signed data files.

### X.2 - Creating a hash (digital fingerprint) for files in a collection

Within a file directory or subtree signed, the `sha256sum` program or
equivalent should be used to create or validate a set of 256 bit SHA
hash codes, saved in a file called `sha256sum.txt`. The `sha256sum` software
is a standard part of the core GNU text utilities, pre-installed or available
on all platforms. The file format consists on one line per file, starting with
the 256 bit hex encoded hash, a space, either space or * for binary, and the
file name including a subdirectory path. The hash values associated with each
file can be compared to track changes in content.

When a data file collection is updated, an existing `sha256sum.txt` file
created with the prior data contents can be renamed to be
`sha256sum.txt.prior` and included in a new `sha256sum.txt` with updated
content.

The commands to rename a prior checksum and signature file, then create
a `sha256sum.txt` for all files in a directory is:

    mv sha256sum.txt sha256sum.txt.prior
    mv sha256sum.txt.sig sha256sum.txt.sig.prior
    sha256sum * >sha256sum.txt

The command to validate hash codes for files in the `sha256sum.txt` is

    sha256sum -c sha256sum.txt

Files with changed content can be identified by comparing the versions,
e.g. with the command:
    diff sha256sum.txt sha256sum.txt.prior
or using a revision control system such as `git`.

Files stored in a `.zip` archive need to be unzipped to use the `sha256sum`
command directly


### X.3 - Creating and verifying a digital signature

A file with name consisting of the data file signed with added `.sig` suffix
is used to hold a detached digital signature. The signed file representing a
collection will be `sha256sum.txt` and the signature for that collection will be
in `sha256sum.txt.sig`. The data format should be the standard OpenPGP
(RFC4880) detached signature file. The recommended software to create
and validate signatures is `gpg2` an implemention of the OpenPGP standard.

The command to create the detached signature of the sha256 file is:

    gpg2 --output sha256sum.txt.sig --detach-sig sha256sum.txt

To verify the signature, use:

    gpg2  --verify sha256sum.txt.sig sha256sum.txt

If there are multiple signature keys, the `--default-key` option can be
used, or edit the `default-key` setting in the `~/.gnupg/gpg.conf` file.

If the `sha256sum.txt` is embedded within a `.zip` file, the signature
can be created with piped commands and added into the zip, e.g.

    unzip -p resultdata.zip sha256sum.txt|gpg2 --output sha256sum.txt.sig --detach-sig
    zip -m resultdata.zip sha256sum.txt.sig

To verify a signature in a zip file, the signature can be unzipped and then
used to compare with the content:

    unzip resultdata.zip sha256sum.txt.sig
    unzip -p resultdata.zip sha256sum.txt|gpg2 --verify sha256sum.txt.sig -

Detached signature files can be used on a file directly, e.g. the
`resultdata.zip` to create `resultdata.zip.sig`, but then the content
of extracted files cannot be individually verified by the SHA256 hash.

### X.4 - Creating additional signatures/endorsements

The originating system should create the `sha256sum.txt.sig` file, but
additional signatures can be collected and added using the file
naming notation `sha256sum.txt.name.sig` where _name_ is a
word identifying the source of the signature.

If the set of files signed is different than a whole collection, the original
`sha256sum.txt` file signed can be renamed to be `sha256sum.txt.name`.

Note: This specification does not include any meanings for the use of the
_name_ used. Added signatures could be used to keep original signatures
imported from an external system, a signature used to indicate _certified_
status, or a notary that provides an external record of a filing.

### X.6 - Creating and managing signature keys

[TODO]

### X.7 - Publishing valid signature keys

[TODO]

### X.7 - Using Hardware Security Modules

[TODO]

