## Open an Issue
All changes should have an associated [Issue](https://github.com/versity/versitygw/issues) to discuss changes or document current deficient behavior. It is recommended to discuss desired changes within an issue to ensure no one is already working on the issue and that the changes would in principle be accepted.

## License/Copyright
All new files in the change should have the versitygw copyright and license headers.  You can add your own copyright as well, but the following is required:
```
// Copyright 2023 Versity Software
// This file is licensed under the Apache License, Version 2.0
// (the "License"); you may not use this file except in compliance
// with the License.  You may obtain a copy of the License at
//
//   http://www.apache.org/licenses/LICENSE-2.0
//
// Unless required by applicable law or agreed to in writing,
// software distributed under the License is distributed on an
// "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
// KIND, either express or implied.  See the License for the
// specific language governing permissions and limitations
// under the License.
```

All changes need to be license compatible with the above license, and should be written by the contributor.  If any 3rd party code needs to be contributed along with the change, discuss with versitygw team in the corresponding issue before opening the PR.

## Run unit tests and make sure all pass
```
go test ./...
```
It is also recommended to add new unit tests when possible to cover new and changed behavior.

## Open GitHub Pull Request
The pull requests will run actions like build, staticcheck, govulncheck, gofmt, etc.  All of these checks should pass before the PR will be considered for merging unless specifically exempted from gateway team.
