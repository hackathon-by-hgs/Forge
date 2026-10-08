# Forge

Forge helps Nigeria's blue-collar gig workers find nearby jobs, clock in with GPS, get paid the moment they clock out, and build a credit record that banks recognise. This root only holds this README and the ignore rules. Each part of the product is its own branch of the same remote, cloned into a folder of the same name.

| Folder | Branch | What it is |
| --- | --- | --- |
| `mobile` | `mobile` | The Flutter worker app for Android and iOS: jobs near you, GPS clock in and out, the wallet and withdrawals, and loans. |
| `frontend` | `frontend` | The Next.js web apps: a public landing site, the employer dashboard and the bank dashboard. |
| `backend` | `backend` | The NestJS API with Prisma and PostgreSQL that all three clients talk to. Payments run through Squad. |

## Setup

```bash
git clone --branch main --single-branch https://github.com/hackathon-by-hgs/Forge.git forge
cd forge
git clone --branch mobile --single-branch https://github.com/hackathon-by-hgs/Forge.git mobile
git clone --branch frontend --single-branch https://github.com/hackathon-by-hgs/Forge.git frontend
git clone --branch backend --single-branch https://github.com/hackathon-by-hgs/Forge.git backend
```

Then follow the README inside each folder. Run git, installs, tests and commits inside the folder that owns the change, never from this root.

## Contributors

Forge was built by the team below. The web apps and the API started as the separate `forge_fe` and `forge_be` repositories, and their history is kept on the `frontend` and `backend` branches.

| Contributor | Worked on |
| --- | --- |
| [Maxima24](https://github.com/Maxima24) | Web apps and API |
| [willy264](https://github.com/willy264) | Web apps |
| [professor-12](https://github.com/professor-12) | Web apps |
| [Ferousco-dev](https://github.com/Ferousco-dev) and [ferousco](https://github.com/ferousco) | Mobile app |
