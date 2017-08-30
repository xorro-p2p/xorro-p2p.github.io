<h2 id="file_sharding">File Sharding and Retrieval</h2>
<p>File sharding is the process of chopping up a file into smaller pieces, naming the pieces, and documenting the files/names in a manifest file such that they can be retrieved and reassembled in the appropriate order.</p>

<p>Sharding improves distribution and reliability of a P2P network, as multiple nodes can store some or all shards of a shared file. If a node containing download information for a shard goes offline, the shard info can be retrieved from a different source.</p>

<p>File sharding also saves bandwidth, where the load of sharing potentially large files is distributed amongst many nodes.</p>

<figure>
  <img src="public/images/file_sharding.png" alt="File sharding">
  <figcaption>The sharding process produces multiple pieces and a manifest file</figcaption>
</figure>

<p>In our implementation, the file adding and sharding process looks like this:</p>
<ul>
  <li>A shared file is added to the network via drag & drop.</li>
  <li>An ID is created for the file by calculating the SHA1 hash of the file's content.</li>
  <li>This ID is repurposed as the filename for the manifest file, with a ".xro" extension. This process is transparent to the user, who only needs to be aware of the ID.</li>
  <li>The original file is sharded into either 1MB chunks, or 50% of the original file size if it is smaller than 1MB.</li>
  <li>Each shard is written to disk - the name of the shard is the SHA1 hash of its content.</li>
  <li>The shards names are written in order to the manifest file, along with the original file name and file size in bytes.</li>
  <li>Manifest and shard location information is broadcasted to peer nodes with the STORE RPC. For each shard/manifest, the hosting node looks in its own routing table for the closest nodes to the ID of the file.  Each of these peer nodes receive a STORE RPC containing the ID and location of the file</li>
</ul>

<p>Below we have the contents of a manifest in JSON format:</p>

```json
  {
    "file_name":"banana.mp4",
    "length":9137395,
    "pieces":[
      "272610812651008498817059664145444816819140431736",
      "255845820650928817902394043384061703021184974492",
      "709124865584808529999187320247131501825035282844",
      "463972141944555281071361859762050722622562309482",
      "665233928362169136349271011451642022996948352498",
      "460767108478119568061824765684889409150273585314",
      "242856439500459087965632547950882773486858003109",
      "1113118586291233368092664992853829437069513635744",
      "55094692080869844054492088107211106202780121432"
    ]
  }
```
<p>File retrieval is handled similarly:</p>
<ul>
  <li>A user enters the ID of the file they want to retrieve from the network. This ID is the hash of the file's content, and also happens to be the name of the manifest file they retrieve first.</li>
  <li>The manifest file is retrieved from the network by issuing FIND VALUE RPCs to peer nodes closest to the ID of the manifest.</li>
  <li>Once the manifest is downloaded, the node repeats the lookup and download process (FIND VALUE RPC) for each shard documented in the manifest.</li>
  <li>After all shards have been downloaded, they are reassembled into the original file.</li>
  <li>The node then broadcasts location info for its newly acquired shards and manifest to peer nodes closest to the respective IDs of the files.</li>
</ul>

<figure>
  <img src="public/images/xorrolores.gif" alt="Retrieving a file from the Xorro P2P Network">
  <figcaption>Retrieving a file from the Xorro P2P Network</figcaption>
</figure>
