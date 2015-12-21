We are using Mocha to run tests, Chai to spy on functions, and Sinon to stub 3rd party functionality at work.  Below are links to each one of the projects.  Please read into their functionality and why or how you would use them before continuing forward.

### Mocha: 
[https://mochajs.org/](https://mochajs.org/)

### Sinon:
[http://sinonjs.org/](http://sinonjs.org/)

### Chai:
[http://chaijs.com/](http://chaijs.com/)

## Writing your unit tests:
  To start a new test create a file called `name_of_file_test.js` inside the tests folder of your repo.  Make sure you keep the same folder structure as you do in the actual codebase.  So for example if your file to test is located in `commands/file_system` you would mirror that structure in the tests folder.

You should also include the file your testing.  So in this example I am testing the folder_command.js file.  I would require this at the top of the test file like so 

`var FolderCommand = require('../../../commands/file_system/folder_command');`  

This gives you the ability to call the functions you will be testing.

  From here you should be setup to start writing your unit tests. Congrats!

## Using Sinon stubs in your test
Let's start with an example.

This is the function we will be testing.

    module.exports.create = function (options) {
      const directory = options.timeStamp + '--' + options.id;

      return Q.nfcall(fs.mkdir, directory);
    };

We want to be able to make sure that `Q.nfcall` was call with specific argument but we dont want `fs.mkdir` to actually make a directory so we use stubs.

Here is what the test would look like. Please disregard the chai.spy.on call until later.

    describe('FolderCommand', function() {
      describe('.create', function() {
        beforeEach(function() {
          Test.sinon.stub(fs, 'mkdir');
          Test.chai.spy.on(Q, 'nfcall');
        });

        it('call Q.nfcall with args', function() {
          FolderCommand.create(options);
          Test.expect(Q.nfcall).to.have.been.called.with(fs.mkdir, directory);
        });
      });
    });

## Using Chai Spys in your test
Let's start with an example.

This is the function we will be testing.

    module.exports.create = function (options) {
      const directory = options.timeStamp + '--' + options.id;

      return Q.nfcall(fs.mkdir, directory);
    };

We want to be able to make sure that `Q.nfcall` was call with specific argument.  To do this we have to spy on the function and then expect it to be called.

Here is what the test would look like.

    describe('FolderCommand', function() {
      describe('.create', function() {
        beforeEach(function() {
          Test.sinon.stub(fs, 'mkdir');
          Test.chai.spy.on(Q, 'nfcall');
        });

        it('call Q.nfcall with args', function() {
          FolderCommand.create(options);
          Test.expect(Q.nfcall).to.have.been.called.with(fs.mkdir, directory);
        });
      });
    });

You can see what in the `beforeEach` we are calling `Test.chai.spy.on(Q, 'nfcall')`.  This allows us to expect the function to be called with the args you are passing in.

## Testing asynchronous code:

This is my example function under test:


    module.exports.create = function (options) {
     const directory = options.timeStamp + '--' + options.id + '/' + options.id + '--' + options.key  + options.ext;

     return Q.nfcall(json2csv, _configureOptions(options.ext, options.data, _reportFields())).then(function(csv) {
       return Q.nfcall(fs.writeFile, directory, csv, 'utf8');
     });
    }

You can see that we are calling `Q.nfcall(json2csv....)` and then when it returns the promise we are calling `Q.nfcall(fs.writeFile.....)`.  Since we are doing this we can't simply call the function and then expect something to happen on the next line.  In our test we will have to call the function and then call `then` on it and expect it to be called there.

Here is an example of how the test would look.

    describe('CsvCommand', function() {
      describe('.create', function() {
        let sandbox;

        beforeEach(function () {
          sandbox = Test.sinon.sandbox.create();
          let stub = sandbox.stub(Q, 'nfcall');
          stub.onFirstCall().returns(Q.resolve(csv));
          stub.onSecondCall();
          sandbox.stub(fs, 'writeFile');
          Test.chai.spy.on(Q, 'nfcall');
        });

        afterEach(function () {
          sandbox.restore();
        });

        it('should call Q.nfcall with fs.writeFile', function(done) {
          CsvCommand.create(options).then(function(data) {
            Test.expect(Q.nfcall).to.have.been.called.with(fs.writeFile, directory, csv, 'utf8');
            done();
          });
        });
      });
    });

When testing promises the `it` block gives you a done argument you can use to tell your tests when the async code is done.  You need to make sure you call that directly after your expect line.

It's important to note that we are using a `sandbox` here to be able to encapsulate this test and clean up the data after the test is run.  If you don't use a `sandbox` then other tests files could potentially fail.

It's also important to note that since we are calling `Q.nfcall` twice (one inside the other).  We have to stub it out twice and define `onFirstCall` and `onSecondCall`.  You will see that `onFirstCall` I am resolving the csv data just like the code does.  Then on the second call is where I am expecting the args to be what they should be. (Took me forever to figure this out....)

Also make sure that you use the afterEach function to restore your sandbox.
