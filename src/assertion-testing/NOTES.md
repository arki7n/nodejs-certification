assert lib methods

� 11:38:17:354 �   [LOG] 1.AssertionError
� 11:38:17:355 �   [LOG] 2.ok

� 11:38:17:355 �   [LOG] 3.equal
� 11:38:17:356 �   [LOG] 4.notEqual

� 11:38:17:356 �   [LOG] 5.deepEqual
� 11:38:17:357 �   [LOG] 7.deepStrictEqual
� 11:38:17:356 �   [LOG] 6.notDeepEqual
� 11:38:17:357 �   [LOG] 8.notDeepStrictEqual

� 11:38:17:357 �   [LOG] 9.strictEqual
� 11:38:17:357 �   [LOG] 10.notStrictEqual

� 11:38:17:358 �   [LOG] 11.throws
� 11:38:17:358 �   [LOG] 12.rejects
� 11:38:17:359 �   [LOG] 13.doesNotThrow
� 11:38:17:359 �   [LOG] 14.doesNotReject

� 11:38:17:359 �   [LOG] 15.ifError
� 11:38:17:360 �   [LOG] 16.match
� 11:38:17:360 �   [LOG] 17.doesNotMatch
� 11:38:17:361 �   [LOG] 18.CallTracker

� 11:38:17:362 �   [LOG] 19.strict // All above method as keys

-------------------------------------------------------------------------------------------------
assert error Object

  � 11:42:32:094 �   [LOG] 0.generatedMessage = true
  � 11:42:32:095 �   [LOG] 1.code = ERR_ASSERTION
  � 11:42:32:095 �   [LOG] 2.actual = 2
  � 11:42:32:096 �   [LOG] 3.expected = 2
  � 11:42:32:096 �   [LOG] 4.operator = strictEqual


-----------------------------------------------
� 11:42:32:048 �   [LOG] 0.fail = function fail(actual, expected, message, operator, stackStartFn) {
  const argsLen = arguments.length;

  let internalMessage = false;
  if (actual == null && argsLen <= 1) {
    internalMessage = true;
    message = 'Failed';
  } else if (argsLen === 1) {
    message = actual;
    actual = undefined;
  } else {
    if (warned === false) {
      warned = true;
      process.emitWarning(
        'assert.fail() with more than one argument is deprecated. ' +
          'Please use assert.strictEqual() instead or only pass a message.',
        'DeprecationWarning',
        'DEP0094'
      );
    }
    if (argsLen === 2)
      operator = '!=';
  }

  if (message instanceof Error) throw message;

  const errArgs = {
    actual,
    expected,
    operator: operator === undefined ? 'fail' : operator,
    stackStartFn: stackStartFn || fail,
    message
  };
  const err = new AssertionError(errArgs);
  if (internalMessage) {
    err.generatedMessage = true;
  }
  throw err;
}
� 11:42:32:063 �   [LOG] 1.AssertionError = class AssertionError extends Error {
  constructor(options) {
    if (typeof options !== 'object' || options === null) {
      throw new ERR_INVALID_ARG_TYPE('options', 'Object', options);
    }
    const {
      message,
      operator,
      stackStartFn,
      details,
      // Compatibility with older versions.
      stackStartFunction
    } = options;
    let {
      actual,
      expected
    } = options;

    const limit = Error.stackTraceLimit;
    Error.stackTraceLimit = 0;

    if (message != null) {
      super(String(message));
    } else {
      if (process.stderr.isTTY) {
        // Reset on each call to make sure we handle dynamically set environment
        // variables correct.
        if (process.stderr.hasColors()) {
          blue = '\u001b[34m';
          green = '\u001b[32m';
          white = '\u001b[39m';
          red = '\u001b[31m';
        } else {
          blue = '';
          green = '';
          white = '';
          red = '';
        }
      }
      // Prevent the error stack from being visible by duplicating the error
      // in a very close way to the original in case both sides are actually
      // instances of Error.
      if (typeof actual === 'object' && actual !== null &&
          typeof expected === 'object' && expected !== null &&
          'stack' in actual && actual instanceof Error &&
          'stack' in expected && expected instanceof Error) {
        actual = copyError(actual);
        expected = copyError(expected);
      }

      if (operator === 'deepStrictEqual' || operator === 'strictEqual') {
        super(createErrDiff(actual, expected, operator));
      } else if (operator === 'notDeepStrictEqual' ||
        operator === 'notStrictEqual') {
        // In case the objects are equal but the operator requires unequal, show
        // the first object and say A equals B
        let base = kReadableOperator[operator];
        const res = StringPrototypeSplit(inspectValue(actual), '\n');

        // In case "actual" is an object or a function, it should not be
        // reference equal.
        if (operator === 'notStrictEqual' &&
            ((typeof actual === 'object' && actual !== null) ||
             typeof actual === 'function')) {
          base = kReadableOperator.notStrictEqualObject;
        }

        // Only remove lines in case it makes sense to collapse those.
        // TODO: Accept env to always show the full error.
        if (res.length > 50) {
          res[46] = `${blue}...${white}`;
          while (res.length > 47) {
            ArrayPrototypePop(res);
          }
        }

        // Only print a single input.
        if (res.length === 1) {
          super(`${base}${res[0].length > 5 ? '\n\n' : ' '}${res[0]}`);
        } else {
          super(`${base}\n\n${ArrayPrototypeJoin(res, '\n')}\n`);
        }
      } else {
        let res = inspectValue(actual);
        let other = inspectValue(expected);
        const knownOperator = kReadableOperator[operator];
        if (operator === 'notDeepEqual' && res === other) {
          res = `${knownOperator}\n\n${res}`;
          if (res.length > 1024) {
            res = `${StringPrototypeSlice(res, 0, 1021)}...`;
          }
          super(res);
        } else {
          if (res.length > 512) {
            res = `${StringPrototypeSlice(res, 0, 509)}...`;
          }
          if (other.length > 512) {
            other = `${StringPrototypeSlice(other, 0, 509)}...`;
          }
          if (operator === 'deepEqual') {
            res = `${knownOperator}\n\n${res}\n\nshould loosely deep-equal\n\n`;
          } else {
            const newOp = kReadableOperator[`${operator}Unequal`];
            if (newOp) {
              res = `${newOp}\n\n${res}\n\nshould not loosely deep-equal\n\n`;
            } else {
              other = ` ${operator} ${other}`;
            }
          }
          super(`${res}${other}`);
        }
      }
    }

    Error.stackTraceLimit = limit;

    this.generatedMessage = !message;
    ObjectDefineProperty(this, 'name', {
      value: 'AssertionError [ERR_ASSERTION]',
      enumerable: false,
      writable: true,
      configurable: true
    });
    this.code = 'ERR_ASSERTION';
    if (details) {
      this.actual = undefined;
      this.expected = undefined;
      this.operator = undefined;
      for (let i = 0; i < details.length; i++) {
        this['message ' + i] = details[i].message;
        this['actual ' + i] = details[i].actual;
        this['expected ' + i] = details[i].expected;
        this['operator ' + i] = details[i].operator;
        this['stack trace ' + i] = details[i].stack;
      }
    } else {
      this.actual = actual;
      this.expected = expected;
      this.operator = operator;
    }
    ErrorCaptureStackTrace(this, stackStartFn || stackStartFunction);
    // Create error message including the error code in the name.
    this.stack; // eslint-disable-line no-unused-expressions
    // Reset the name.
    this.name = 'AssertionError';
  }

  toString() {
    return `${this.name} [${this.code}]: ${this.message}`;
  }

  [inspect.custom](recurseTimes, ctx) {
    // Long strings should not be fully inspected.
    const tmpActual = this.actual;
    const tmpExpected = this.expected;

    if (typeof this.actual === 'string') {
      this.actual = addEllipsis(this.actual);
    }
    if (typeof this.expected === 'string') {
      this.expected = addEllipsis(this.expected);
    }

    // This limits the `actual` and `expected` property default inspection to
    // the minimum depth. Otherwise those values would be too verbose compared
    // to the actual error message which contains a combined view of these two
    // input values.
    const result = inspect(this, {
      ...ctx,
      customInspect: false,
      depth: 0
    });

    // Reset the properties after inspection.
    this.actual = tmpActual;
    this.expected = tmpExpected;

    return result;
  }
}
� 11:42:32:066 �   [LOG] 2.ok = function ok(...args) {
  innerOk(ok, args.length, ...args);
}
� 11:42:32:066 �   [LOG] 3.equal = function equal(actual, expected, message) {
  if (arguments.length < 2) {
    throw new ERR_MISSING_ARGS('actual', 'expected');
  }
  // eslint-disable-next-line eqeqeq
  if (actual != expected && (!NumberIsNaN(actual) || !NumberIsNaN(expected))) {
    innerFail({
      actual,
      expected,
      message,
      operator: '==',
      stackStartFn: equal
    });
  }
}
� 11:42:32:067 �   [LOG] 4.notEqual = function notEqual(actual, expected, message) {
  if (arguments.length < 2) {
    throw new ERR_MISSING_ARGS('actual', 'expected');
  }
  // eslint-disable-next-line eqeqeq
  if (actual == expected || (NumberIsNaN(actual) && NumberIsNaN(expected))) {
    innerFail({
      actual,
      expected,
      message,
      operator: '!=',
      stackStartFn: notEqual
    });
  }
}
� 11:42:32:067 �   [LOG] 5.deepEqual = function deepEqual(actual, expected, message) {
  if (arguments.length < 2) {
    throw new ERR_MISSING_ARGS('actual', 'expected');
  }
  if (isDeepEqual === undefined) lazyLoadComparison();
  if (!isDeepEqual(actual, expected)) {
    innerFail({
      actual,
      expected,
      message,
      operator: 'deepEqual',
      stackStartFn: deepEqual
    });
  }
}
� 11:42:32:068 �   [LOG] 6.notDeepEqual = function notDeepEqual(actual, expected, message) {
  if (arguments.length < 2) {
    throw new ERR_MISSING_ARGS('actual', 'expected');
  }
  if (isDeepEqual === undefined) lazyLoadComparison();
  if (isDeepEqual(actual, expected)) {
    innerFail({
      actual,
      expected,
      message,
      operator: 'notDeepEqual',
      stackStartFn: notDeepEqual
    });
  }
}
� 11:42:32:068 �   [LOG] 7.deepStrictEqual = function deepStrictEqual(actual, expected, message) {
  if (arguments.length < 2) {
    throw new ERR_MISSING_ARGS('actual', 'expected');
  }
  if (isDeepEqual === undefined) lazyLoadComparison();
  if (!isDeepStrictEqual(actual, expected)) {
    innerFail({
      actual,
      expected,
      message,
      operator: 'deepStrictEqual',
      stackStartFn: deepStrictEqual
    });
  }
}
� 11:42:32:069 �   [LOG] 8.notDeepStrictEqual = function notDeepStrictEqual(actual, expected, message) {
  if (arguments.length < 2) {
    throw new ERR_MISSING_ARGS('actual', 'expected');
  }
  if (isDeepEqual === undefined) lazyLoadComparison();
  if (isDeepStrictEqual(actual, expected)) {
    innerFail({
      actual,
      expected,
      message,
      operator: 'notDeepStrictEqual',
      stackStartFn: notDeepStrictEqual
    });
  }
}
� 11:42:32:070 �   [LOG] 9.strictEqual = function strictEqual(actual, expected, message) {
  if (arguments.length < 2) {
    throw new ERR_MISSING_ARGS('actual', 'expected');
  }
  if (!ObjectIs(actual, expected)) {
    innerFail({
      actual,
      expected,
      message,
      operator: 'strictEqual',
      stackStartFn: strictEqual
    });
  }
}
� 11:42:32:071 �   [LOG] 10.notStrictEqual = function notStrictEqual(actual, expected, message) {
  if (arguments.length < 2) {
    throw new ERR_MISSING_ARGS('actual', 'expected');
  }
  if (ObjectIs(actual, expected)) {
    innerFail({
      actual,
      expected,
      message,
      operator: 'notStrictEqual',
      stackStartFn: notStrictEqual
    });
  }
}
� 11:42:32:072 �   [LOG] 11.throws = function throws(promiseFn, ...args) {
  expectsError(throws, getActual(promiseFn), ...args);
}
� 11:42:32:072 �   [LOG] 12.rejects = async function rejects(promiseFn, ...args) {
  expectsError(rejects, await waitForActual(promiseFn), ...args);
}
� 11:42:32:072 �   [LOG] 13.doesNotThrow = function doesNotThrow(fn, ...args) {
  expectsNoError(doesNotThrow, getActual(fn), ...args);
}
� 11:42:32:073 �   [LOG] 14.doesNotReject = async function doesNotReject(fn, ...args) {
  expectsNoError(doesNotReject, await waitForActual(fn), ...args);
}
� 11:42:32:073 �   [LOG] 15.ifError = function ifError(err) {
  if (err !== null && err !== undefined) {
    let message = 'ifError got unwanted exception: ';
    if (typeof err === 'object' && typeof err.message === 'string') {
      if (err.message.length === 0 && err.constructor) {
        message += err.constructor.name;
      } else {
        message += err.message;
      }
    } else {
      message += inspect(err);
    }

    const newErr = new AssertionError({
      actual: err,
      expected: null,
      operator: 'ifError',
      message,
      stackStartFn: ifError
    });

    // Make sure we actually have a stack trace!
    const origStack = err.stack;

    if (typeof origStack === 'string') {
      // This will remove any duplicated frames from the error frames taken
      // from within `ifError` and add the original error frames to the newly
      // created ones.
      const tmp2 = StringPrototypeSplit(origStack, '\n');
      ArrayPrototypeShift(tmp2);
      // Filter all frames existing in err.stack.
      let tmp1 = StringPrototypeSplit(newErr.stack, '\n');
      for (const errFrame of tmp2) {
        // Find the first occurrence of the frame.
        const pos = ArrayPrototypeIndexOf(tmp1, errFrame);
        if (pos !== -1) {
          // Only keep new frames.
          tmp1 = ArrayPrototypeSlice(tmp1, 0, pos);
          break;
        }
      }
      newErr.stack =
        `${ArrayPrototypeJoin(tmp1, '\n')}\n${ArrayPrototypeJoin(tmp2, '\n')}`;
    }

    throw newErr;
  }
}
� 11:42:32:077 �   [LOG] 16.match = function match(string, regexp, message) {
  internalMatch(string, regexp, message, match);
}
� 11:42:32:078 �   [LOG] 17.doesNotMatch = function doesNotMatch(string, regexp, message) {
  internalMatch(string, regexp, message, doesNotMatch);
}
� 11:42:32:082 �   [LOG] 18.CallTracker = class CallTracker {

  #callChecks = new SafeSet()

  calls(fn, exact = 1) {
    if (process._exiting)
      throw new ERR_UNAVAILABLE_DURING_EXIT();
    if (typeof fn === 'number') {
      exact = fn;
      fn = noop;
    } else if (fn === undefined) {
      fn = noop;
    }

    validateUint32(exact, 'exact', true);

    const context = {
      exact,
      actual: 0,
      // eslint-disable-next-line no-restricted-syntax
      stackTrace: new Error(),
      name: fn.name || 'calls'
    };
    const callChecks = this.#callChecks;
    callChecks.add(context);

    return function() {
      context.actual++;
      if (context.actual === context.exact) {
        // Once function has reached its call count remove it from
        // callChecks set to prevent memory leaks.
        callChecks.delete(context);
      }
      // If function has been called more than expected times, add back into
      // callchecks.
      if (context.actual === context.exact + 1) {
        callChecks.add(context);
      }
      return ReflectApply(fn, this, arguments);
    };
  }

  report() {
    const errors = [];
    for (const context of this.#callChecks) {
      // If functions have not been called exact times
      if (context.actual !== context.exact) {
        const message = `Expected the ${context.name} function to be ` +
                        `executed ${context.exact} time(s) but was ` +
                        `executed ${context.actual} time(s).`;
        ArrayPrototypePush(errors, {
          message,
          actual: context.actual,
          expected: context.exact,
          operator: context.name,
          stack: context.stackTrace
        });
      }
    }
    return errors;
  }

  verify() {
    const errors = this.report();
    if (errors.length > 0) {
      throw new AssertionError({
        message: 'Function(s) were not called the expected number of times',
        details: errors,
      });
    }
  }
}
� 11:42:32:084 �   [LOG] 19.strict = function strict(...args) {
  innerOk(strict, args.length, ...args);
}
