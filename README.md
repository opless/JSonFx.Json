JSonFx.Json
===========

A Unity3d compatible JsonFx.Json fork



I use this with the following interface and implementation...
(You'll obviously have to make your own plain old C# objects to serialise/deserialise)


```
/// the ISerializer implementation

using System;

namespace SimonWaite.Serialization
{
	public interface ISerializer
	{
		/// <summary>
		/// Stringify Encode the specified obj.
		/// </summary>
		/// <param name='obj'>
		/// Object to serialse.
		/// </param>
		string Encode (object obj);
		/// <summary>
		/// Decode the specified json and success.
		/// </summary>
		/// <param name='serialised'>
		/// Stringified serialised data.
		/// </param>
		/// <param name='success'>
		/// Success.
		/// </param>
		/// <typeparam name='T'>
		/// The type to deserialise.
		/// </typeparam>
		T Decode<T> (string serialised, ref bool success) where T : class;
	}
}
```

```
//
// JsonFxSerializer.cs
//
// Author:
//       simon <simon@simonwaite.com>
//
// Copyright (c) 2012 Simon Waite All Rights Reserved
//
using System;
using System.IO;
using System.Text;
using SimonWaite.IoC;
using JsonFx.Json;

namespace SimonWaite.Serialization
{
    // Jsonfx nabbed from: https://bitbucket.org/darktable/jsonfx-for-unity3d/src
    // has problems with Dictionary<int, string> :(
    [Exports(typeof(ISerializer))]
    public class JsonFxSerializer : ISerializer
    {
        public JsonFxSerializer()
        {
        }
        private const string TypeHintName ="__type";
        #region encode/decode
        public string Encode (object obj)
        {
            /*
            var settings = new JsonSerializerSettings ();
            settings.TypeNameHandling = TypeNameHandling.Auto;
            settings.Formatting = Formatting.Indented;
            return JsonConvert.SerializeObject (obj, settings);
            */
            var settings = new JsonWriterSettings();
            settings.PrettyPrint = true;
            settings.Tab = "  ";
            settings.MaxDepth=100;
            settings.TypeHintName = TypeHintName;
           
//            settings.UseXmlSerializationAttributes = true;
            var sb = new StringBuilder();
            var wr = new JsonWriter(sb,settings);
            wr.Write(obj);
            return sb.ToString();
        }
        
        public T Decode<T>(string json, ref bool success) where T : class
        {
            /*
            T ret = null;
            try {
                var settings = new JsonSerializerSettings ();
                settings.TypeNameHandling = TypeNameHandling.Auto;
                ret = JsonConvert.DeserializeObject<T> (json, settings);
                success = true;
            } catch (Exception ex) {
                ret = null;
                success = false;
            }
            return ret;
            */
            T ret = null;
            try
            {
                var settings = new JsonReaderSettings();
                settings.AllowUnquotedObjectKeys = true;
                settings.AllowNullValueTypes = true;
                settings.TypeHintName= TypeHintName;

                var rd = new JsonReader(json,settings);
                ret = rd.Deserialize<T>();
                success = true;
            } catch (Exception ex)
            {
                ret = null;
                success = false;
            }
            return ret;
        }
        #endregion
    }
}
```
