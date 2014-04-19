#grunt-port-pick
[![Build Status](https://travis-ci.org/devaos/grunt-port-pick.svg?branch=master)](https://travis-ci.org/devaos/grunt-port-pick) [![Dependency Status](https://david-dm.org/devaos/grunt-port-pick.svg?theme=shields.io)](https://david-dm.org/devaos/grunt-port-pick) [![devDependency Status](https://david-dm.org/devaos/grunt-port-pick/dev-status.svg?theme=shields.io)](https://david-dm.org/devaos/grunt-port-pick#info=devDependencies) [![Coverage Status](https://coveralls.io/repos/devaos/grunt-port-pick/badge.png)](https://coveralls.io/r/devaos/grunt-port-pick)

##Installation

Install npm package:

```bash
npm install grunt-port-pick --save-dev
```

Add the following to your project's `Gruntfile` in order to load the task:

```js
grunt.loadNpmTasks('grunt-port-pick');
```

##Usage Examples

Tired of manually configuring ports only to have them conflict?  This task will scan for an unused port, given a range, and swap in the unused port into other tasks' configurations.

###A basic example showing simple port picking:

```js
// Project configuration.
grunt.initConfig({
  portPick: {
    karma: {
      targets: [
        'karma.options.port'
      ]
    }
  },

  karma: {
    options: {
      port: -1
    }
  }
});

grunt.registerTask('test', [ 'portPick', 'karma' ]);
```

###A complex example showing various ways to use selected ports

```js
// Project configuration.
grunt.initConfig({
  // Specify what ports need to be injected where and what additional ports
  // need to be allotted.
  portPick: {
    options: {
      port: 7654,
      extra: 1
    },
    selenium: {
      targets: [
        'selenium.options.port',
        'protractor.test1.args.seleniumPort',
        'protractor.test2.args.seleniumPort'
      ]
    },
    parallelFuncTest1: {
      options: {
        name: 'port-pick-connect1'
      },
      targets: [
        'connect.test1.port',
      ]
    },
    parallelFuncTest2: {
      options: {
        name: 'port-pick-connect2'
      },
      targets: [
        'connect.test2.port',
      ]
    }
  },

  // Sample selenium runner.
  selenium: {
    options: {
      port: -1
    },
  },

  // Sample grunt-connect target to emulate two parallelly running web
  // servers where the ports are dynamically set.
  connect: {
    test1: {
      port: -1
    },
    test2: {
      port: -1
    }
  },

  // Sample grunt-protractor-runner target to emulate two e2e runners hitting
  // the two connect web servers started above.
  protractor: {
    test1: {
      options: {
        args: {
          seleniumPort: -1,
          baseUrl: 'http://localhost:<%= grunt.config.get("port-pick-connect1") %>'
        }
      }
    },
    test2: {
      options: {
        args: {
          seleniumPort: -1,
          baseUrl: 'http://localhost:<%= grunt.config.get("port-pick-connect2") %>'
        }
      }
    }
  },

  // Sample grunt-karma target to emulate the karma port selection for running
  // unit tests alongside with our e2e tests above, using the "extraPorts"
  // option to have the port available via grunt templates.
  karma: {
    parallel: {
      options: {
        port: '<%= grunt.config.get("port-pick-1") %>'
      }
    }
  },

  parallel: {
    options: {
      stream: true
    },
    tests: {
      tasks: [{
        grunt: true,
        args: [
          'portPick',
          'unit',
          'func1',
          'func2',
          '--portPickUsePorts=<%= grunt.config.get("port-pick-used") %>']
      }]
    },
});

grunt.registerTask('func1', [ 'portPick:parallelFuncTest1', 'connect:test1', 'protractor:test1' ]);
grunt.registerTask('func2', [ 'portPick:parallelFuncTest1', 'connect:test2', 'protractor:test2' ]);
grunt.registerTask('unit', [ 'karma:parallel' ]);

grunt.registerTask('test', [ 'portPick', 'selenium', 'parallel:tests' ]);
```

##Options

1. `port` -- The port to start scanning from, inclusive, for the first available port to use.  Defaults to 8765.
2. `limit` -- Do not scan for a port after this one, fail if one is not found between `port` and `limit`.  Defaults to false, port-pick will scan all the way up to 65535.
3. `extra` -- Set the number of extra port-pick-# configurations to supply.  Defaults to 0.  If set to greater than 0, it will populate <%= port-pick-1 %>, <%= port-pick-2 %>, etc.
4. `hostname` -- The IP/hostname to bind to when checking for available ports.  Defaults to 0.0.0.0, so binds to all interfaces.
5. `name` -- Set the property name for port-pick to use when picking a port for a target.  Defaults to undefined where it will not populate a separate configuration with the target's selected port.

##Command line options

1. `--portPickUsePorts=#,#,...` -- A comma delimited list of ports to select from, instead of scanning for new unused ports.  This is useful if you need to provide the ports that were selected to spawned grunt tasks, such as when using [grunt-parallel](https://github.com/iammerrick/grunt-parallel) or [grunt-concurrent](https://github.com/sindresorhus/grunt-concurrent).  Defaults to undefined.
