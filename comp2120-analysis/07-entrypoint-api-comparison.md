
============================================================================
## environment
============================================================================
  python:                                        3.13.16 (main, Oct 14 2026, 00:55:38) [Clang 23.1.3 ]
  importlib.metadata:                            available
  pkg_resources:                                 /Users/comp6120/a5/experiments/venvs/api-compare/lib/python3.13/site-packages/pkg_resources/__init__.py
  setuptools:                                    81.0.0

============================================================================
## 1. group selection: which entry points come back for group='opsdroid_connectors'
============================================================================
  importlib count:                               3
  importlib names:                               ['fixtureconn', 'fixtureconn', 'zzz_dup_only']
  pkg_resources count:                           3
  pkg_resources names:                           ['fixtureconn', 'fixtureconn', 'zzz_dup_only']
  same name set:                                 True
  same ORDER:                                    True
  importlib order:                               ['fixtureconn', 'fixtureconn', 'zzz_dup_only']
  pkg_resources order:                           ['fixtureconn', 'fixtureconn', 'zzz_dup_only']

============================================================================
## 2. the three attributes/methods opsdroid actually uses
============================================================================
  importlib ep.name:                             'fixtureconn'
  importlib ep.value:                            'opsdroid_fixture_connector.connector'
  importlib ep.group:                            'opsdroid_connectors'
  importlib type(ep):                            EntryPoint
  importlib ep.load():                           <module 'opsdroid_fixture_connector.connector' from '/Users/comp6120/a5/experiments/venvs/api-compare/lib/python3.13/site-packages/opsdroid_fixture_connector/connector.py'>
  importlib has .dist:                           True
  importlib has .module_name:                    False
  importlib has .attrs:                          False

  importlib ep.name:                             'fixtureconn'
  importlib ep.value:                            'opsdroid_fixture_dup:ConnectorDup'
  importlib ep.group:                            'opsdroid_connectors'
  importlib type(ep):                            EntryPoint
  importlib ep.load():                           <class 'opsdroid_fixture_dup.ConnectorDup'>
  importlib has .dist:                           True
  importlib has .module_name:                    False
  importlib has .attrs:                          False

  importlib ep.name:                             'zzz_dup_only'
  importlib ep.value:                            'opsdroid_fixture_dup:ConnectorDup'
  importlib ep.group:                            'opsdroid_connectors'
  importlib type(ep):                            EntryPoint
  importlib ep.load():                           <class 'opsdroid_fixture_dup.ConnectorDup'>
  importlib has .dist:                           True
  importlib has .module_name:                    False
  importlib has .attrs:                          False

  pkg_resources ep.name:                         'fixtureconn'
  pkg_resources ep.module_name:                  'opsdroid_fixture_connector.connector'
  pkg_resources ep.attrs:                        ()
  pkg_resources type(ep):                        EntryPoint
  pkg_resources ep.load():                       <module 'opsdroid_fixture_connector.connector' from '/Users/comp6120/a5/experiments/venvs/api-compare/lib/python3.13/site-packages/opsdroid_fixture_connector/connector.py'>
  pkg_resources has .value:                      False
  pkg_resources has .group:                      False

  pkg_resources ep.name:                         'fixtureconn'
  pkg_resources ep.module_name:                  'opsdroid_fixture_dup'
  pkg_resources ep.attrs:                        ('ConnectorDup',)
  pkg_resources type(ep):                        EntryPoint
  pkg_resources ep.load():                       <class 'opsdroid_fixture_dup.ConnectorDup'>
  pkg_resources has .value:                      False
  pkg_resources has .group:                      False

  pkg_resources ep.name:                         'zzz_dup_only'
  pkg_resources ep.module_name:                  'opsdroid_fixture_dup'
  pkg_resources ep.attrs:                        ('ConnectorDup',)
  pkg_resources type(ep):                        EntryPoint
  pkg_resources ep.load():                       <class 'opsdroid_fixture_dup.ConnectorDup'>
  pkg_resources has .value:                      False
  pkg_resources has .group:                      False


============================================================================
## 3. the exact expression opsdroid.loader uses: {ep.name: ep for ep in ...}
============================================================================
  importlib dict keys:                           ['fixtureconn', 'zzz_dup_only']
    fixtureconn ->:                              opsdroid_fixture_dup:ConnectorDup
    zzz_dup_only ->:                             opsdroid_fixture_dup:ConnectorDup
  pkg_resources dict keys:                       ['fixtureconn', 'zzz_dup_only']
    fixtureconn ->:                              opsdroid_fixture_dup.ConnectorDup
    zzz_dup_only ->:                             opsdroid_fixture_dup.ConnectorDup
    winner for 'fixtureconn' identical?:         True
    winner for 'zzz_dup_only' identical?:        True

============================================================================
## 4. construction signature (what the unit test relies on)
============================================================================
  importlib EntryPoint signature:                (name: 'str', value: 'str', group: 'str') -> 'None'
  pkg_resources EntryPoint signature:            (name: 'str', module_name: 'str', attrs: 'Iterable[str]' = (), extras: 'Iterable[str]' = (), dist: 'Distribution | None' = None) -> 'None'
  test uses `dist=` kwarg:                       pkg_resources.EntryPoint(name, value, dist=dist) only
  importlib accepts dist= kwarg:                 NO -> TypeError: EntryPoint.__init__() got an unexpected keyword argument 'dist'

============================================================================
## 5. does `entry_points()` warn or fail when setuptools provides the metadata?
============================================================================
  warnings raised:                               none

============================================================================
## 6. entry points in a directory added to sys.path AFTER the metadata import
============================================================================
  why this matters:                              opsdroid/loader.py:451 appends DEFAULT_MODULE_DEPS_PATH to sys.path,
  :                                              then loader.py:456 enumerates entry points in the same call
  installed opsdroid-fixture-late into:          /var/folders/lq/b43r_dw15hd8_bq89qjjbvv00000gn/T/latepath-2a1mxtyl
  target contains:                               ['opsdroid_fixture_late', 'opsdroid_fixture_late-0.0.1.dist-info']
  importlib     before append:                   ['fixtureconn', 'fixtureconn', 'zzz_dup_only']
  pkg_resources before append:                   ['fixtureconn', 'fixtureconn', 'zzz_dup_only']
  importlib     after  append:                   ['fixtureconn', 'fixtureconn', 'lateconn', 'zzz_dup_only']
  pkg_resources after  append:                   ['fixtureconn', 'fixtureconn', 'zzz_dup_only']
  importlib sees the late distribution:          True
  pkg_resources sees it without add_entry:       False
  pkg_resources after working_set.add_entry:     ['fixtureconn', 'fixtureconn', 'lateconn', 'zzz_dup_only']
  ...and even then:                              True

============================================================================
## 6b. two distributions with the same project name on sys.path
============================================================================
  why this matters:                              a same-name shadow copy on sys.path could, in principle, change which entry points are found
  method:                                        each scenario runs in a fresh interpreter (metadata state is process-global)
  second copy of opsdroid-fixture-dup installed into: /var/folders/lq/b43r_dw15hd8_bq89qjjbvv00000gn/T/duppath-i8rfu74d
  importlib / pkg_resources WITHOUT the shadow copy: 3 / 3
  importlib / pkg_resources WITH the shadow copy: 3 / 3
  importlib is unaffected by the shadow copy:    True
  pkg_resources is affected by the shadow copy:  False
  interpretation:                                no difference here: both de-duplicate by project name + version, so a same-name shadow copy adds no entry points

============================================================================
## 7. extras syntax in an entry point value
============================================================================
  note:                                          pkg_resources and importlib.metadata both support 'mod:attr [extra]'
  opsdroid does not use extras in entry points:  grep confirms no '[' in values

============================================================================
## done
============================================================================
