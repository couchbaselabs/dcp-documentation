# UPR Implementation Notes

For keeping track of how we actually plan to satisfy the requirements we
laid out in the UPR spec.

## Snapshot/deduping

### Barrier Points

To provide snapshot markers, we can mantain a "barrier point" that we
prevent deduplication to happen behind. That is, if the barrier point is
is 400, we do not delete items from memory that are before sequence 400.
Once clients receive sequence 400, they are also sent the snapshot
marker message. Since this provides a guarantee that there are no
missing keys caused by deduplication behind a point, this barrier point
can only be *created* at the *most recent* sequence number.

When a satisfactory number of clients have seen it, or based on some
other logic, we can destroy the barrier point, clean up the data it was
preventing us from deduping, and then create a new one. Except for
extreme cases, we should probably try to make sure that persistence is
always getting snapshot markers.



