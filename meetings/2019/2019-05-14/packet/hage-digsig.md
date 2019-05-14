CH: 3/12/2019 - Digital Signature Draft Proposal for Discussion

## 6. Recommendations

### 6.11. Security

#### 6.11.1. Digital Signatures

Digital signatures are ubiquitous in the software development
community, but currently are virtually unused in securing election data.
A new election system developed for the future must incorporate digital
signatures to ensure the integrity and validity of election data.

Manufacturers that work to ensure the quality of their products address
quality within the whole supply chain. With election data security,
it is likewise important to make use of digital signatures to ensure
integrity, validity, and traceability across the whole chain of source
input data, derived data, configuration files, software, and output data.

Besides digital signatures used to authenticate and secure election
data, a primary basis of data security should be from posting the complete
chain of non-confidential data on a network-accessible server with open
access and subject to public scrutiny.

The following requirements are guidelines for using digital signatures
for data and software (a sample implementation is described in Appendix X) :

* All election-related data transferred between computers must have a
  digital signature that can be used to validate the integrity and origin
  of exchanged data. (This includes data archived for backup.)

* The process of creating digital signatures and how to authenticate
  signatures must be documented.

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
      signature. A public record of the signatures issued by the notary server
      can be used to confirm the time and content of digitally notarized data.

    * Data exported from one infosystem might have a digital signature created
      by the hardware in the source system, then imported into another
      infosystem. After importing, an additional digital signature created
      by the importing system hardware could be added. (The importing system
      might convert the original "primary" signature to a secondary/endorsement
      with the new signature created by the importing system acting as a new
      "primary" signature.)

* All software used should have digital signatures attached and validated.

    * Computer systems must use a secure boot to authenticate software initially
      loaded on startup, and to authenticate the base operating system and
      secured data collection. Ideally, the secure boot would require a special
      digital signature, e.g. from a state voting certification key, and the
      key required could not be changed without replacing the hardware.

    * Software packaged into containers should have digital signatures associated
      with the container file and validated at least upon system startup.

    * Software collections (e.g. the complete set of source and library files
      used by an application) should have an associated hash for each file
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
  hash (digital fingerprint) of input/configuration files, software collection,
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
      data file. (Sometimes binary database files can change even
      when data inside is unchanged, and individual differences cannot be
      tracked in the binary database file.)

## Appendix X - Sample Digital Signature Protocol

This appendix contains a sample implementation of the digital signature
recommended requirements given above in section 6.11.1.

This implementation is used in the OSVTAC
sample data and application repositories as a demonstration. It can be
used as-is or as the basis to develop an alternative/improved protocol.

The `gpgsign` python script available in the
[OSVTAC/crypto-tools github repository] is a wrapper that invokes the `gpg`
and `sha256sum` commands in order to perform the operations described below.

The `gpgsign` can create or authenticate a digital signature and can validate
the contents of a collection of files in a directory or zip file. See the
documentation in the repository for usage information.

### X.1 - Data file collections

A collection of data files is stored in a file system, and can be signed as
a set with a single digital signature. All files in a directory can be considered
to be the collection, or a subset of files in a directory can be in a collection
with an explicit list of file names defining the subset. Files in a directory
tree can be in a single collection signed at the top level, or each subdirectory
can have independent signatures.

Since a digital signature "signs" a given single piece of data, a 2-step
system is needed to work with collections of files:

* A file is created with the SHA256 hash (digital fingerprint) for each file
in the collection. The hash codes validate the contents of each individual file.

* A digital signature (or multiple signatures) authenticates the hash file.

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

For a file directory or subtree to be signed, the `sha256sum` program or
equivalent should be used to create or validate a set of 256 bit SHA
hash codes, saved in a file called `SHA256SUMS.txt`. The `sha256sum` software
is a standard part of the core GNU text utilities, pre-installed or available
on all platforms. The file format consists on one line per file, starting with
the 256 bit hex encoded hash, a space, either space or * for binary, and the
file name including a subdirectory path. The hash values associated with each
file can be compared to track changes in content.

When a data file collection is updated, an existing `SHA256SUMS.txt` file
created with the prior data contents can be renamed to be
`SHA256SUMS.txt.prior` and included in a new `SHA256SUMS.txt` with updated
content.

The commands to rename a prior checksum and signature file, then create
a `SHA256SUMS.txt` for all files in a directory is:

    mv SHA256SUMS.txt SHA256SUMS.txt.prior
    mv SHA256SUMS.txt.sig SHA256SUMS.txt.sig.prior
    sha256sum * >SHA256SUMS.txt

The command to validate hash codes for files in the `SHA256SUMS.txt` is

    sha256sum -c SHA256SUMS.txt

Files with changed content can be identified by comparing the versions,
e.g. with the command:
    diff SHA256SUMS.txt SHA256SUMS.txt.prior
or using a revision control system such as `git`.

Note: Files stored in a `.zip` archive need to be unzipped to use the `sha256sum`
command directly.


### X.3 - Creating and verifying a digital signature

A file with name consisting of the data file signed with added `.sig` suffix
is used to hold a detached digital signature. The signed file representing a
collection will be `SHA256SUMS.txt` and the signature for that collection will be
in `SHA256SUMS.txt.sig`. The data format should be the standard OpenPGP
(RFC4880) detached signature file. The recommended software to create
and validate signatures is `gpg2` an implemention of the OpenPGP standard.

The command to create the detached signature of the sha256 file is:

    gpg2 --output SHA256SUMS.txt.sig --detach-sig SHA256SUMS.txt

To verify the signature, use:

    gpg2  --verify SHA256SUMS.txt.sig SHA256SUMS.txt

If there are multiple signature keys, the `--default-key` option can be
used, or edit the `default-key` setting in the `~/.gnupg/gpg.conf` file.

If the `SHA256SUMS.txt` is embedded within a `.zip` file, the signature
can be created with piped commands and added into the zip, e.g.

    unzip -p resultdata.zip SHA256SUMS.txt|gpg2 --output SHA256SUMS.txt.sig --detach-sig
    zip -m resultdata.zip SHA256SUMS.txt.sig

To verify a signature in a zip file, the signature can be unzipped and then
used to compare with the content:

    unzip resultdata.zip SHA256SUMS.txt.sig
    unzip -p resultdata.zip SHA256SUMS.txt|gpg2 --verify SHA256SUMS.txt.sig -

Detached signature files can be used on a file directly, e.g. the
`resultdata.zip` to create `resultdata.zip.sig`, but then the content
of extracted files cannot be individually verified by the SHA256 hash.

Note: The commands above to validate signatures is embedded within the `gpgsign`
script.

### X.4 - Creating additional signatures/endorsements

The originating system should create the `SHA256SUMS.txt.sig` file, but
additional signatures can be collected and added using the file
naming notation `SHA256SUMS.txt.name.sig` where _name_ is a
word identifying the source of the signature.

Note: This specification does not include any meanings for the use of the
_name_ used. Added signatures could be used to keep original signatures
imported from an external system, a signature used to indicate _certified_
status, or a notary that provides an external record of a filing.

### X.5 - Signing/Verifying a subset of files

If the set of files to be signed is different than the whole collection
of files in a directory (or zip file), a suffix in the file name
should be used to label a subset, e.g. `SHA256SUMS-suffix.txt` where _suffix_
is the label indicating a subset.

If a signed collection of data is copied from another location and combined
with other files in the same directory, the original `SHA256SUMS.txt`
and `SHA256SUMS.txt.sig` should be renamed with a suffix. For example,
if importing a set of signed data created on an external system:

    unzip ~/imports/ems.zip
    mv SHA256SUMS.txt SHA256SUMS-ems.txt
    mv SHA256SUMS.txt.sig SHA256SUMS-ems.txt.sig

By renaming the original `SHA256SUMS` with the suffix `-ems`, the subset of
files in the `ems` collection is identified and the signature file with that
suffix is associated the the subset collection.

Note: a suffix could be always used in the `SHA256SUMS` files if file directories
contain multiple collections.

### X.6 - Signing git commits

If the `git` system is used to track changes in data files or software,
or `github.com` is used to distribute data files, an additional signature
should be used to
[sign git commits][https://help.github.com/en/articles/signing-commits].
\[A git _commit_ sets a specific version of data updates prior to transfer,
or for computing differences.]

For each commit, the `-S` command option should be used, e.g.

    git commit -S -m 'Final certified results'

The git system will then invoke `gpg` to include a digital signature for this
commit.

If a git tag is used to mark a significant release of the data, a signature
can be added and
[associated with the tag][https://help.github.com/en/articles/signing-tags]
using the `-s` option, e.g.

    git tag -s 'v2.0.0' -m 'Beta release of version 2'

To verify the signature associated with a tag, use:

    git tag -v 'v2.0.0'

The signature associated with the commit or tag authenticates the collection
of data in that commit. For each commit, git can identify any differences in
file data between the current state and the state when the commit was made.
When downloading data, git can verify the downloaded data matches the signed
commit data.

To use signatures with git, it should be
[configured to set the signing key][https://help.github.com/en/articles/telling-git-about-your-signing-key],
e.g.

    git config --global user.signingkey AABBCCDD

where `AABBCCDD` is the GPG key ID to use. (The `--global` sets this for
all repositories on the system. If omitted, it only applies to the current
repository.)

To enable signing all commits by default (for a specific repository), use:

    git config commit.gpgsign true


Note: The `SHA256SUMS.txt` and files and `.sig` signatures allow applications to
authenticate content and track changes in individual files, independent of
the version control or distribution system. The separate and additional
signature with a git commit facilitates the usual verifications when
downloading updates with git.

### X.6 - Creating and managing signature keys

[TODO]

### X.7 - Publishing valid signature keys

[TODO]

### X.8 - Using Hardware Security Modules

All digital signatures used for election related data and software should
use a hardware device (HSM) to store the secret keys. This could be from
a built-in hardware chip such as the TPM (Trusted Platform Module) that
plugs into a laptop or desktop motherboard, or from a dongle that plugs into
a USB port. The removable USB dongles can be used for special signatures made
by an authorized official, then stored securely when not in use. For signatures
added after each computation step in processing election data, a built-in
device might be better.

Described below are some notes on how to use a few specific kinds of HSMs
available.

#### X.8.1 - TPM (Trustem Platform Module)

The [TPM][https://en.wikipedia.org/wiki/Trusted_Platform_Module] is a standard
cryptographic device installed within a computer to perform a number of
cryptographic operations, including encryption, password protection, and
digital signatures.

[TODO]

#### X.8.2 - Yubikey USB security key

[Yubico][https://www.yubico.com/] makes a number of USB devices that can be used
to generate digital signatures, and can also be used for secure 2-factor
authentication for logging into computer systems or web sites. Two types
of devices are available, one that can be put on a keychain and temporarily inserted,
and another low-profile version that can be kept in a USB port. There is
a FIPS certified version, though this is not the latest model.

[TODO]


