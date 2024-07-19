# Introduction to Active Directory
### Active Directory
- Directory service used to manage Windows networks.
- Stores information about objects on the network and makes the objects easily available to users and admins.
- **Everything in Active Directory considered as an object.**
- **Active directory enables centralized, secure management of an entire network, which might span a building, a city or multiple locations throughout the world**.
### Active Directory - Components
- Different type of objects can interoperate using active directory.
- **Schema defines** objects and their attributes.
- **Query and index mechanism** provides searching and publication of objects and their properties.
- **Global catalog** contains information about every object in the directory.
- **Replication service** distributes information across domain controllers.

### Active Directory - Structure
- Each domain must be in a forest, even there is a single domain in a forest.
- Forests, domains and organization units(OUs) are the basic building blocks of any active directory structure.
- A forest may contain multiple domains.
- A domain may contain multiple OUs.
- Forest is considered as security boundary in Active Directory environment.

- **Because a forest considered as security boundary in Active directory**, any domain including child domains compromising will be resulted as all forest compromising.
- Within a forest, all domains has built-in trust for each other.

