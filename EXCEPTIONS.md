# Room exceptions (temporary)

See **subx-factory EXCEPTIONS.md E22**.

This room carries a temporary `factory.js` `?auth=` join deep-link (`authQ` + `onAuthStateChanged`, shell v1.2.8) ahead of the next factory stamp. `FACTORY.lock` `files.factory.js` is the sha256 of the room file after that patch (same shape stamp.py writes). Remove this file and re-stamp after [subx-factory#47](https://github.com/jebbdykstra99/subx-factory/pull/47) merges.
