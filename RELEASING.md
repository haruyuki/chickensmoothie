# Publishing with GitHub Trusted Publishing

The workflow `.github/workflows/publish.yml` builds and tests the wheel, checks
both distributions with Twine, then uploads through PyPI Trusted Publishing.
No PyPI API tokens or GitHub repository secrets are needed. Twine remains a
local validation tool, not the authentication mechanism.

## One-time setup

1. Commit and push the workflow, tests, and packaging files to `main` in
   `haruyuki/chickensmoothie`. The workflow must exist on the default branch
   before GitHub shows its manual Run workflow button.
2. In GitHub repository Settings > Environments, create `testpypi` and `pypi`.
   Configure a required reviewer for `pypi` where supported. For a solo project,
   allow self-review so you can approve your own releases.
3. Create/verify your accounts and enable two-factor authentication on PyPI and
   TestPyPI. They are separate accounts.
4. At https://test.pypi.org/manage/account/publishing/, add a pending GitHub publisher:
   - PyPI project name: `chickensmoothie`
   - Owner: `haruyuki`
   - Repository name: `chickensmoothie`
   - Workflow name: `publish.yml` (filename only)
   - Environment name: `testpypi`
5. At https://pypi.org/manage/account/publishing/, enter the same values, except
   environment name must be `pypi`.

If the project already exists under your account, add the publisher from that
project's Manage > Publishing page instead. A pending publisher creates the
project on first upload; it does not reserve the package name.

## Test a release

1. Commit the intended version in `pyproject.toml`; run `uv lock` after changing it.
2. Push the commit. Open GitHub Actions > Publish package > Run workflow.
3. Select your branch or tag, choose target `testpypi`, and run it.
4. Review the successful run and https://test.pypi.org/project/chickensmoothie/.
5. Verify installation in a separate local environment:

```powershell
uv venv .venv-testpypi
uv pip install --python .venv-testpypi/Scripts/python.exe "aiohttp>=3.13.5" "lxml>=6.0.4" "cssselect>=1.4.0"
uv pip install --python .venv-testpypi/Scripts/python.exe --index-url https://test.pypi.org/simple/ --no-deps chickensmoothie==0.1.0
.venv-testpypi/Scripts/python.exe -I -m unittest discover -s tests -v
```

Dependencies come from real PyPI; only this package comes from TestPyPI.
Replace `0.1.0` with your release version. Repeating a successful upload of the
same filenames fails; use a new version for changed releases.

## Publish to PyPI

For version `0.1.0`, tag the exact tested commit and push the tag:

```powershell
git tag v0.1.0
git push origin v0.1.0
```

In GitHub Actions > Publish package > Run workflow, select tag `v0.1.0` under
Use workflow from, choose target `pypi`, and run. Approve the `pypi` environment
when prompted. The workflow rejects production runs unless the selected tag
matches `v` plus the version in `pyproject.toml`.

The workflow rebuilds the selected commit for each run; TestPyPI and PyPI runs
are not guaranteed to produce byte-identical archives. Neither ordinary pushes
nor creating a tag alone publishes anything: publication requires a manual run.

After success, users can run `python -m pip install chickensmoothie`.
For later releases, increase the version, test, create a matching tag, and repeat.
Already published distribution filenames cannot be reused, even after deletion.

## Local checks (no upload)

```powershell
uv sync --locked
uv run python -m unittest discover -s tests -v
uv build --no-sources
uvx twine check --strict dist/chickensmoothie-0.1.0-py3-none-any.whl dist/chickensmoothie-0.1.0.tar.gz
```

Use only the files for the intended release when checking locally.
GitHub builds start in a fresh checkout and upload only that run's artifacts.

## References

- https://docs.pypi.org/trusted-publishers/
- https://docs.pypi.org/trusted-publishers/creating-a-project-through-oidc/
- https://docs.pypi.org/trusted-publishers/using-a-publisher/
