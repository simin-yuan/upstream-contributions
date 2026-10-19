Adds greencheck to the Development Tools section.

Disclosure: this is my own tool and it is brand new (published on PyPI, 1 GitHub star). greencheck tests whether a validator, gate, or metric can actually distinguish the cases it claims to: deliberately damage an artefact, re-run the check, and report the mutations that escape, plus flag measurements that cannot react differently to inputs they claim to distinguish. For JSON Schema in particular, a constraint that is silently never enforced (for example a keyword ignored under the declared draft) is exactly the kind of failure it surfaces.

Changes: data.yaml (new entry) and README.md (the rendered bullet inserted in its sorted position, between Sourcemeta Studio and schema-envoy). I updated the README by hand to mirror the generator output; happy to run the generator instead if you prefer.

Scope note: greencheck is a general validator-testing tool rather than a JSON Schema implementation, so if it does not fit this list, feel free to close.