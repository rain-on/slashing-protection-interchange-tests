# Slashing Protection Interchange Tests (EIP-3076)

Tests for EIP-3076:

https://eips.ethereum.org/EIPS/eip-3076

Discussion:

https://ethereum-magicians.org/t/eip-3076-validator-client-interchange-format-slashing-protection/4883

## How to run

Each test directory contains an interchange file and some extra data about how to test it.

For example:

```json
{
  "name": "single_validator_genesis_attestation",
  "genesis_validators_root": "0x0000000000000000000000000000000000000000000000000000000000000000",
  "steps": [
    {
      "should_succeed": true,
      "allow_partial_import": false,
      "interchange": {
        "metadata": {
          "interchange_format_version": "5",
          "genesis_validators_root": "0x0000000000000000000000000000000000000000000000000000000000000000"
        },
        "data": [
          {
            "pubkey": "0xa99a76ed7796f7be22d5b7e85deeb7c5677e88e511e0b337618f8c4eb61349b4bf2d153f649f7b53359fe8b94a38e44c",
            "signed_blocks": [],
            "signed_attestations": [
              {
                "source_epoch": "0",
                "target_epoch": "0"
              }
            ]
          }
        ]
      },
      "blocks": [],
      "attestations": [
        {
          "pubkey": "0xa99a76ed7796f7be22d5b7e85deeb7c5677e88e511e0b337618f8c4eb61349b4bf2d153f649f7b53359fe8b94a38e44c",
          "source_epoch": "0",
          "target_epoch": "0",
          "should_succeed": false
        }
      ]
    }
  ]
}
```

To run a test, first initialize a new (empty) slashing protection database.

Then for each entry in `steps`, import the `interchange`, process the `blocks` and `attestations`,
and continue to the next step.

Determine the test outcome according to the meanings of each of the fields,
which are as follows:

* `name: string`: the name of the test-case, informational.
* `genesis_validators_root: Root`: the genesis validators root to use when
  creating the empty slashing protection database, or to compare the import
  against.
* `steps[i].should_succeed: bool`: whether the `steps[i].interchange` given is valid and should
  be imported successfully.
* `steps[i].allow_partial_import: bool`: whether the `steps[i].interchange` contains some
  slashable data with respect to itself or the existing contents of the database, and therefore
  a partial import is justified.
* `steps[i].interchange: Interchange`: slashing protection interchange data as described
  by the spec.
* `steps[i].blocks: [object]`: a list of block signings to be attempted **after**
  importing the `interchange`, detailed below.
* `steps[i].attestations: [object]`: a list of attestation signings to be attempted **after**
  importing the `interchange`, detailed below.

Each block in `blocks` is structured as:

```json
{
  "pubkey": "0xa99a76ed7796f7be22d5b7e85deeb7c5677e88e511e0b337618f8c4eb61349b4bf2d153f649f7b53359fe8b94a38e44c",
  "slot": 1,
  "signing_root": "0x0000000000000000000000000000000000000000000000000000000000000000",
  "should_succeed": false
}
```

Your test-runner should attempt to sign a block with `signing_root` at the
given slot from the given `pubkey`. The `should_succeed` field describes
whether this signing should be accepted (true) or rejected (false). Clients
using a slashing protection mechanism that admits false positives may consider
a rejection as success even if `should_succeed` is true.

Each attestation in `attestations` is structured as:

```json
{
  "pubkey": "0xa99a76ed7796f7be22d5b7e85deeb7c5677e88e511e0b337618f8c4eb61349b4bf2d153f649f7b53359fe8b94a38e44c",
  "source_epoch": 11,
  "target_epoch": 12,
  "signing_root": "0x0000000000000000000000000000000000000000000000000000000000000000",
  "should_succeed": true
}
```

Similarly to above, your test-runner should attempt to sign an attestation with these parameters
using the given `pubkey`, and succeed based on the value of `should_succeed`. Again, false positives
may be treated as successes.

Note that the top-level `genesis_validators_root` is not necessarily the same
as the GVR contained in the interchange, to allow us to test the case where
they are mismatched.

## Downloading the tests

For the time being you can hit this URL to get a .tar.gz of this repo:

```
https://github.com/eth2-clients/slashing-protection-interchange-tests/tarball/<COMMIT_HASH>
```

Or you can use a git submodule.
