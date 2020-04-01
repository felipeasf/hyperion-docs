#### Detailed description of the ecosystem.config.js

````javascript
const {addApiServer, addIndexer} = require("./definitions/ecosystem_settings");

module.exports = {
    apps: [
        addIndexer('eos'),
        addApiServer('eos', 1)
    ]
};
````