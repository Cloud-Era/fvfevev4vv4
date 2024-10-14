Fetching package.json and eon_id for Eaton-Vance-Corp/xfs-angular-frontend... Traceback (most recent call last):

File "C:\Users\VGudigopuram\Documents\SBOM_API\Hector\github_report.py", line 100, in <module>

asyncio.run(main()) File "C:\Program Files\Python312\Lib\asyncio\runners.py", line 194, in run return runner.run(main)

File "C:\Program Files\Python312\Lib\asyncio\runners.py", line 118, in run return self._loop.run_until complete (task)

File "C:\Program Files\Python312\Lib\asyncio\base_events.py", line 687, in run_until_complete return future.result()

File "C:\Users\VGudigopuram\Documents\SBOM_API\Hector\github_report.py", line 79, in main package_json = await get_package_json(session, ORG, REPO)

File "C:\Users\VGudigopuram\Documents\SBOM_API\Hector\github_report.py", line 36, in get package_json package_json_content json.loads(content['content'])

File "C:\Program Files\Python312\Lib\json\__init__.py", line 346, in loads return _default_decoder.decode(s)

File "C:\Program Files\Python312\Lib\json\decoder.py", line 337, in decode

obj, end self.raw_decode (s, idx=_w(s, 0).end())

raise JSONDecodeError ("Expecting value", s, err.value) from None json.decoder.JSONDecodeError: Expecting value: line 1 column 1 (char 0)

File "C:\Program Files\Python312\Lib\json\decoder.py

", line 355, in raw decode
