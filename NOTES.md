https://github.com/oguimbal/pgsql-ast-parser/issues/8

function normalize(query) {
    let i = 0;
    const byName = {};
    const normalized = query.replace(/(?<!:):(\w+)\b/gi, (_, x) => {
      if (byName[x]) {
        return '$' + byName[x];
      }
      return '$' + (byName[x] = ++i)
    });

    const keys = Object.keys(byName);
    function toArgList(args) {
      const ret = Array(keys.length);
      for (const k of keys) {
        ret[byName[k] - 1] = args[k];
      }
      return ret;
    }
  return {normalized, toArgList};
}

// transform a named query to a standard positioned parameters query
const {normalized, toArgList} = normalize('select * from xxx where id = :id AND other=:other + :id');

console.log(normalized); // 'select * from xxx where id = $1 AND other=$2 + $1'
console.log(toArgList({id: 'myId', other: 42 })); // [ 'myId', 42 ]

// this is parsable !
parse(normalized);

npm install pgsql-parser
